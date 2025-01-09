# Maximizing Multer: Advanced Configuration and Tips for Powerful File Uploads

Many of you might be using multer to store the files uploaded by the user first in your server and then in some cloud to provide atomicity in your web applications. For this you might be using multer or some of you might be uploading the files directly in some cloud storage like Azure Blob or AWS buckets.

When using multer to store files by the user directly in your server, do y'all as backend devs really trust the users? Will the incoming file always be just an image, video, or pdf? What if it is an .exe, .MSI, or .jar? Or if it is so large that it overwhelms your server?
////


## Storage
You might be familiar with the storage options multer provides. But there is more to it.

```javascript
import multer from "multer";

// DiskStorage
const storage = multer.diskStorage({
    destination: function (req, file, cb) {
        cb(null, "./public/temp")
    },
    filename: function (req, file, cb) {
        cb(null, `${Date.now()}-${file.originalname}`)
    }
})

/*
----- OR ----- MemoryStorage------

const storage = multer.memoryStorage()

*/

export const upload = multer({storage})
```

This is what a basic multer middleware looks like. However more can be added to this to limit and filter your uploads.

The **file** is an object provided by multer which is used for handling the multipart/form-data. When a file is uploaded, Multer processes it and makes it available in the **req** object in your route handler.

The file contains information like :

| Key          | Description                                | Note          |
|--------------|--------------------------------------------|---------------|
| fieldname    | Field name specified in the form            |               |
| originalname | Name of the file on the user’s computer     |               |
| encoding     | Encoding type of the file                  |               |
| mimetype     | Mime type of the file                      |               |
| size         | Size of the file in bytes                  |               |
| destination  | The folder to which the file has been saved | DiskStorage   |
| filename     | The name of the file within the destination | DiskStorage   |
| path         | The full path to the uploaded file          | DiskStorage   |
| buffer       | A Buffer of the entire file                | MemoryStorage |


This will be useful in setting up later configurations for your uploads.

## fileFilter
Set this to a function to control which files should be uploaded and which should be skipped. The function should look like this:
```javascript
import path from "path"

const fileFilter = (req, file, cb) => {
// The extensions you want to allow
    const allowedExtensions = [
        '.jpeg', '.jpg', '.png', '.gif', '.bmp', '.tiff', '.svg', 
        '.mp4', '.mov', '.mkv', '.avi', '.webm', '.3gp',
        '.pdf', '.doc', '.docx', '.epub', '.tar', '.zip', '.rar',
        '.mp3', '.wav', '.ogg', '.aac',
        '.txt', '.html', '.json', '.xml',
        '.gz', '.bz2',
        '.7z', '.iso'
    ]
// The mimetypes for the extensions you want to allow
    const allowedMimeTypes = [
        'image/jpeg', 'image/png', 'image/gif', 'image/bmp', 'image/tiff', 'image/svg+xml',
        'video/mp4', 'video/quicktime', 'video/x-matroska', 'video/avi', 'video/webm', 'video/3gpp',
        'application/pdf', 'application/msword', 'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
        'application/epub+zip', 'application/x-tar', 'application/zip', 'application/x-rar-compressed',
        'audio/mpeg', 'audio/wav', 'audio/ogg', 'audio/aac',
        'text/plain', 'text/html', 'application/json', 'application/xml',
        'application/gzip', 'application/x-bzip2',
        'application/x-7z-compressed', 'application/x-iso9660-image'
    ]

// get the extension and mimetype of the file
    const extname = path.extname(file.originalname).toLowerCase()
    const mimetype = file.mimetype;

// Check if the extname and mimetype is valid
    const isExtensionValid = allowedExtensions.includes(extname)
    const isMimetypeValid = allowedMimeTypes.includes(mimetype)

// return callback if both are valid
    if (isExtensionValid && isMimetypeValid) {
        return cb(null, true)
    } else {
        return cb(new Error('Unsupported file type'), false)
    }

}
```

So what's going on?

- Create an array for the **extensions** you want to allow
- Create another array for the **mimetypes** you want to allow
  why mimes? when we can check for .exe files in extensions?
  we will discuss this in the later part of this blog.
- Get the extension of the file with the help of **path** module in node.js.
  **path** module helps in handling the path of the files
  **eg. dirname/filename.txt**
- Get the MIME of the file
- the file object contains this info which was discussed the the above part
  Check weather the **extname** and **mimetype** of the file matches any values in the allowed extension and mime array.
-  If both are present return the callback to indicate weather it is to be accepted or rejected.

## What are MIMES ? Why are they required?
MIME (Multipurpose Internet Mail Extensions) are a standard way of identifying and handling the document based on the content in it. MIME types consist of a primary type and a sub-type, separated by a slash (e.g., text/plain, application/pdf, video/mp4).

## Importance:
- **Accuracy**: MIME types give a more accurate description of the file types than extensions.

- **Security**: Relying solely on file extensions is not sufficient for identifying file types because a file with a .jpg extension could actually be an executable or malware. MIME types provide an additional layer of identification by examining the file's content. However, MIME types alone are also not foolproof. It's important to use a combination of file extensions, MIME type checking, and content analysis to ensure accurate identification and to enhance security.
- **Web and Network Protocols**: MIME types are used in HTTP headers, email, and other network protocols to properly handle and display files. For example, a web server might send a file with Content-Type: image/jpeg to tell the browser that the file should be displayed as an image.

 **Correct Handling:**
 - **File Extension:**.jpeg
 - **MIME Type:** image/jpeg
 - **Usage:** The browser or application processes it as an image.

**Potential Issue:**
 - **File Extension:**.jpeg
 - **MIME Type:** application/pdf (incorrect)
 - **Usage:** The application might try to open the file as a PDF, leading to errors or incorrect handling.

## limits
An object specifying the size limits of the following optional properties.

| Key           | Description                                      | Default   |
|---------------|--------------------------------------------------|-----------|
| fieldNameSize | Max field name size                              | 100 bytes |
| fieldSize     | Max field value size (in bytes)                  | 1 MB      |
| fields        | Max number of non-file fields                    | Infinity  |
| fileSize      | For multipart forms, the max file size (in bytes) | Infinity  |
| files         | For multipart forms, the max number of file fields| Infinity  |
| parts         | For multipart forms, the max number of parts (fields + files) | Infinity |
| headerPairs   | For multipart forms, the max number of header key=>value pairs to parse | 2000 |

```javascript
limits: { 
        fileSize: 1024 * 1024 * 50 
    }
```

1 MB = 1024 * 1024 bytes
Therefore 50 MB is the size limit set for a file .

```javascript

export const upload = multer({
    storage,
    fileFilter,
    limits: { 
        fileSize: 1024 * 1024 * 50 
    }
})
```

So this is how we can use multer to the fullest and elevate or backend applications.
I came to know this while I was building project and thought it was worth sharing.





