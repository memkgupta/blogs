---
title: "Building a Seamless File Upload System: Handling Chunked Uploads And Large File Uploads"
seoTitle: "Efficient Chunked and Large File Uploads"
seoDescription: "Learn to create a file upload system with chunked uploads, handling large files efficiently with Node.js streams and HTTP streaming"
datePublished: Sat Nov 23 2024 07:31:27 GMT+0000 (Coordinated Universal Time)
cuid: cm3tuq79b000d09jv9nxx7jyl
slug: building-a-seamless-file-upload-system-handling-chunked-uploads-and-large-file-uploads

---

Hello guys in this article we will be exploring how can we create a file upload system like AWS ( joking guys 😂) , but we will look how we can implement a functionality of uploading large files using chunked uploads , so let’s get started.

### **But before starting , why chunked uploads , why not simple file upload using ?**

1. Uploading a large file in a single request can lead to timeout issues or increased memory usage on the client and server.
    
2. If the file is too large let’s say 1GB , then it is obviously not a good approach to send all the 1GB file data in a single request , the server will be having really bad time if we do so 🙂
    
3. If the upload fails midway (due to network issues or server errors), the entire file needs to be reuploaded, wasting bandwidth and time.
    
4. Difficult to provide meaningful feedback to users because the entire file is uploaded in one go.
    

So now we know why we can’t upload large files in a single request , hence what we will be doing is sending our file in parts to the server and server will keep writing the file with our chunks uploaded.

### Key Concepts to Understand Before We Begin

Before going to the implementation part, we should know some concepts and don’t worry if you don’t know them , I am here 😎

### 1 . HTTP Streaming

HTTP Streaming refers to the process of transmitting data over HTTP as a continuous stream rather than sending it all at once in a single response or request. This technique is commonly used for scenarios like video/audio streaming, live feeds, large file uploads, or real-time data delivery.

In typical HTTP request server terminates the connection but here connection is not terminated which allows continuous data communication between server and client.

We can declare a HTTP request as stream by using some headers

* Transfer-Encoding : setting this header to “chunked”
    
* Content-length : this header is not defined in stream requests so that server can decide that the coming request is a stream request
    
* Content-type : We also use “application/octet-stream” in Content-Type header to declare that the mime-type of coming data is not specified and it is binary data
    

### 2 . Streams in NodeJS

In Node.js, **streams** are a powerful way to handle data that is being read from or written to external sources like files, network connections, or standard input/output. They allow you to process large chunks of data piece by piece instead of loading the entire content into memory at once, making it more efficient for handling large amounts of data.

### **Types of Streams in Node.js**

Node.js streams are divided into four main types, based on how the data flows:

1. **Readable Streams**
    
    * These streams allow you to read data from a source.
        
    * Examples: `fs.createReadStream()`, HTTP request objects, process.stdin.
        
    * **Methods**:
        
        * `read()`: Reads data from the stream.
            
        * `on('data', callback)`: Listens for chunks of data being read.
            
        * `pipe()`: Pipes the stream to a writable stream.
            
2. **Writable Streams**
    
    * These streams allow you to write data to a destination.
        
    * Examples: `fs.createWriteStream()`, HTTP response objects, process.stdout.
        
    * **Methods**:
        
        * `write()`: Writes data to the stream.
            
        * `end()`: Ends the writable stream after all data has been written.
            
3. **Duplex Streams**
    
    * These streams can both read from and write to a source. They are a combination of readable and writable streams.
        
    * Example: `net.Socket` (used in network communication).
        
    * **Methods**:
        
        * `read()`, `write()`, `on('data')`, `pipe()`.
            
4. **Transform Streams**
    
    * These are a special kind of duplex stream where the data that is written to the stream is transformed before it is read.
        
    * Example: `zlib.createGzip()` (used for compression).
        
    * **Methods**:
        
        * `transform()` is typically used for processing and modifying data.
            

For now we will only use read and write stream.

**<mark>One cool thing about Streams is the piping </mark> we may not use it for now but i will tell you about it any ways**

It is the feature provided by Streams API allowing us to pipe the output of one stream to the input of another stream . It’s like how pipelines are set up , basically data (water) is coming from one stream (pipe) and that incoming data (water) is streamed (piped) into another stream ( connected pipe)

And One more cool thing is that the **<mark>request and response in node js are also Streams</mark>** so we will be using streams to implement chunked uploads in our server

### 3 . Chunk Uploads

Chunked uploads involve dividing a file into smaller pieces (chunks) and sending these chunks to the server separately. Each chunk is uploaded one by one, and the server appends that chunk into the original file. This approach has several benefits:

* **Resumability**: If an upload is interrupted (e.g., due to network failure), the upload can be resumed from the last successfully uploaded chunk, rather than starting from scratch.
    
* **Progress Tracking**: Since chunks are smaller and discrete, the progress of each chunk can be tracked, providing feedback to users on the upload status.
    
* **Efficient Memory Usage**: Rather than holding the entire file in memory, chunked uploads allow for memory-efficient handling of large files. The server only processes one chunk at a time, reducing the load on memory and CPU.
    

### 4 . **HTTP Headers for Metadata**

In chunked file uploads, HTTP headers play a key role in managing and tracking the state of the upload. Some common headers used in chunked uploads include:

* `chunkNumber`: Identifies the current chunk being uploaded. The server can use this to order and track which chunk of the file is being uploaded at any given time.
    
* `uploadKey`: A unique identifier that associates all chunks with a specific upload session. This key helps the server track which chunks belong to which file upload, especially in cases where multiple files are being uploaded concurrently.
    

We can also provide more headers for authentication purpose like a token for validating the session.

## So now we will be discussing the flow of our application

1. First the client will initialize the upload by sending all the file metadata and will recieve the uploadKey and a token for the session.
    
2. As the server recieves the initialize request , it will create a upload key and the directory with the same upload key ( this will be unique) and also saves the metadata in the database.And in response the uploadKey and token is sent to the client.
    
3. Client will start sending chunks to the server. The request will be containing upload-key and chunk-number , token in it’s headers
    
4. The Server will verify the headers and will create a empty file with that metadata and will start listening the \`on\` event of the request. And the recieved chunks will be appended to the file created.
    
5. Now the subsequent chunks will be transferred over this connection by the client and the server will append those chunks into our file.
    
6. And after all the chunks are uploaded A url or a id pointing towards the database record of that file will be sent to the client for the file retrieval.
    

Now I think we are ready to code , so let’s go

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1732344529312/e6db44e6-e4e2-4fda-9adb-8dd6465cc766.gif align="center")

Express App

```javascript
// index.js
require('dotenv').config();
const express = require('express');
const routes = require('./router.js');
const connect = require('./db');
const app = express();

app.use(express.json());
app.use(express.urlencoded({ extended: true }));

connect();

app.use('/', express.static('./public'));
app.use('/files', routes);

app.listen(3343, () => {
    console.log('Server running on port 3343');
}); 
```

A function to connect to the database (I am using mongoDB)

```javascript
const mongoose = require('mongoose');

const connect = async () => {
    try {
        await mongoose.connect(process.env.DB_URI);
    } catch (error) {
        console.error('Database connection error:', error);
        process.exit(1);
    }
};

module.exports = connect;
```

FileStorage model for mongoose

```javascript
const { Schema, default: mongoose } = require("mongoose");

const schema = new Schema({
    key: {type:String},
   fileName: {type:String},
    size: {type:Number},
    path: {type:String},
  
    status: {type:String},
    metadata:{type:Map},
    uploadedChunks: [Number]
  })
  const FileStorage = mongoose.model("FileStorage",schema);
  module.exports = FileStorage
```

Routes for handling api requests

```javascript
const express = require('express');
const router = express.Router();
const StorageService = require('./StorageService.js');
const path = require('path');
const FileStorage = require('./model.js');
const fs = require('fs');

// Initialize StorageService
const fileStorage = new StorageService();

// Initialize upload route
router.post('/upload/initialize', async (req, res) => {
  try {
    const { fileName, fileSize, metadata } = req.body;

    // Validate request body
    if (!fileName || !fileSize) {
      return res.status(400).json({ error: 'File name and size are required' });
    }

    const upload = await fileStorage.initializeUpload(fileName, fileSize, metadata);
    res.status(201).json({
      uploadKey: upload,
      status: 'ready_to_upload'
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Upload chunk route
router.post('/upload/chunk', async (req, res) => {
  try {
    const { id, chunknumber } = req.headers;

    // Validate headers
    if (!id || chunknumber === undefined) {
      return res.status(400).json({ error: 'Missing required headers' });
    }

    const fileMetadata = await FileStorage.findById(id);
    if (!fileMetadata) {
      return res.status(404).json({ error: 'Upload not initialized' });
    }

    const filePath = path.join(fileMetadata.path, fileMetadata.fileName);
    // Ensure the file exists before writing chunks
    if (!fs.existsSync(filePath)) {
      fs.writeFileSync(filePath, ''); // Create an empty file if not exists
    }

    const fileStream = fs.createWriteStream(filePath, {
      flags: 'r+',
      start: chunknumber * 1024 * 1024 // 1MB chunk size
    });

    // Write chunk data to the file
    await new Promise((resolve, reject) => {
      req.on('data', (chunk) => {
        fileStream.write(chunk, (err) => {
          if (err) reject(err);
        });
      });

      req.on('end', async () => {
        try {
          fileMetadata.uploadedChunks = fileMetadata.uploadedChunks || [];
          fileMetadata.uploadedChunks.push(parseInt(chunknumber));
          await fileMetadata.save();
          resolve();
        } catch (err) {
          reject(err);
        }
      });

      req.on('error', reject);
    });

    res.status(200).json({
      message: 'Chunk uploaded successfully',
      chunkNumber: chunknumber
    });
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

// Finalize upload route
router.post('/upload/finalize', async (req, res) => {
  try {
    const { uploadKey } = req.body;
    if (!uploadKey) {
      return res.status(400).json({ error: 'Upload key is required' });
    }

    const finalizedUpload = await fileStorage.finalizeUpload(uploadKey);
    res.status(200).json({
      uploadKey: finalizedUpload.key,
      status: 'completed',
      fileMetadata: finalizedUpload
    });
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});
module.exports = router;
```

StorageService Class

```javascript
const path = require('path');
const fs = require('fs');
const FileStorage = require('./model.js');
const crypto = require('crypto'); // Make sure to require crypto if using it

class StorageService {
    constructor(uploadDir = "./upload") {
        this.uploadDir = uploadDir;
    }

    async initializeUpload(fileName, fileSize, userId, metadata = {}) {
        const uniqueName = this.generateUniqueFileName(fileName);
        const filePath = path.join(this.uploadDir, uniqueName);

        // Ensure the directory exists
        if (!fs.existsSync(filePath)) {
            fs.mkdirSync(filePath, { recursive: true });
        }

        const fileMetadata = await FileStorage.create({
            key: uniqueName,
            fileName,
            size: fileSize,
            path: filePath,
            userId,
            status: 'uploading',
            metadata,
            uploadedChunks: []
        });

        return fileMetadata._id;
    }

    async finalizeUpload(id) {
        const fileMetadata = await FileStorage.findById(id);
        fileMetadata.status = 'completed';
        return await fileMetadata.save();
    }

    generateUniqueFileName(originalName) {
        const timestamp = Date.now();
        const randomString = crypto.randomUUID().toString('hex');
        const extension = path.extname(originalName);
        return `${timestamp}-${randomString}${extension}`;
    }
}

module.exports = StorageService;
```

Static index.html which will be our client

```xml
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Large File Upload</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 500px;
            margin: 0 auto;
            padding: 20px;
        }
        #progress-container {
            width: 100%;
            background-color: #f0f0f0;
            border-radius: 5px;
            margin-top: 10px;
        }
        #progress-bar {
            width: 0%;
            height: 20px;
            background-color: #4CAF50;
            border-radius: 5px;
            transition: width 0.3s;
        }
    </style>
</head>
<body>
    <div>
        <input type="file" id="fileInput">
        <button id="uploadButton">Upload File</button>
        <div id="progress-container">
            <div id="progress-bar"></div>
        </div>
        <div id="status"></div>
    </div>

    <script>
        class FileUploader {
            constructor(apiBaseUrl) {
                this.apiBaseUrl = apiBaseUrl;
                this.chunkSize = 1 * 1024 * 1024; // 1MB
            }

            async uploadLargeFile(file) {
                const totalChunks = Math.ceil(file.size / this.chunkSize);
                const initResponse = await this.initializeUpload(file);
                const uploadKey = initResponse.uploadKey;

                for (let chunk = 0; chunk < totalChunks; chunk++) {
                    const start = chunk * this.chunkSize;
                    const end = Math.min(start + this.chunkSize, file.size);
                    const chunkData = file.slice(start, end);

                    await this.uploadChunk(uploadKey, chunk, chunkData);
                    this.updateProgressBar((chunk + 1) / totalChunks * 100);
                }

                await this.finalizeUpload(uploadKey);
            }

            async initializeUpload(file) {
                const response = await fetch(`${this.apiBaseUrl}/upload/initialize`, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({
                        fileName: file.name,
                        fileSize: file.size,
                        metadata: {
                            type: file.type,
                            lastModified: file.lastModified
                        }
                    })
                });
                return response.json();
            }

            async uploadChunk(uploadKey, chunkNumber, chunkData) {
               
                await fetch(`${this.apiBaseUrl}/upload/chunk`, {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/octet-stream',
                        'id': uploadKey,
                        'chunknumber': chunkNumber
                    },
                    body: chunkData
                });
            }


            async finalizeUpload(uploadKey) {
                await fetch(`${this.apiBaseUrl}/upload/finalize`, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ uploadKey })
                });
            }

            updateProgressBar(progress) {
                const progressBar = document.getElementById('progress-bar');
                const statusDiv = document.getElementById('status');
                
                progressBar.style.width = `${progress}%`;
                statusDiv.textContent = `Uploading: ${Math.round(progress)}%`;
            }
        }

        document.getElementById('uploadButton').addEventListener('click', async () => {
            const fileInput = document.getElementById('fileInput');
            const file = fileInput.files[0];
            
            if (!file) {
                alert('Please select a file');
                return;
            }

            const uploader = new FileUploader('/files');
            
            try {
                await uploader.uploadLargeFile(file);
                alert('Upload Complete!');
            } catch (error) {
                console.error('Upload failed:', error);
                alert('Upload Failed');
            }
        });
    </script>
</body>
</html>
```

For now i have only implemented Uploading file but you can also implement file fetching using streams in similar way and you should definitely try it .

### DEMO

%[https://youtu.be/lTwW2DkWNmg] 

Hope you liked it if any advice , query please drop it down in the comments .  
Thank you. 🙏