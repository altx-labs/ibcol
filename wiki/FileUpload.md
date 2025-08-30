# File Upload System

The IBCOL platform includes a robust file upload system for handling user-submitted documents such as whitepapers, presentations, student ID cards, and transcripts. This document explains how the file upload system works.

## File Upload Architecture

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│                 │     │                 │     │                 │
│  FilePond       │────▶│  Micro Service  │────▶│  Google Cloud   │
│  Component      │     │  Endpoint       │     │  Storage        │
│                 │     │                 │     │                 │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

The file upload system consists of:

1. **FilePond Component**: Client-side file upload interface
2. **Micro Service Endpoint**: Server-side handler for file uploads
3. **Google Cloud Storage**: Cloud storage for uploaded files
4. **File Reference System**: Storing file references in the database

## FilePond Integration

The application uses the [FilePond](https://pqina.nl/filepond/) library for file uploads:

```jsx
import { FilePond, registerPlugin } from 'react-filepond';
import FilePondPluginFileValidateType from 'filepond-plugin-file-validate-type';
import FilePondPluginImageExifOrientation from 'filepond-plugin-image-exif-orientation';
import FilePondPluginImagePreview from 'filepond-plugin-image-preview';
import FilePondPluginFileValidateSize from 'filepond-plugin-file-validate-size';

// Register plugins
registerPlugin(
  FilePondPluginFileValidateType,
  FilePondPluginFileValidateSize,
  FilePondPluginImageExifOrientation,
  FilePondPluginImagePreview
);

// In a component:
<FilePond
  allowMultiple={false}
  acceptedFileTypes="application/pdf, application/zip"
  labelFileTypeNotAllowed="Only PDF and ZIP files are allowed"
  allowFileSizeValidation={true}
  maxTotalFileSize="500MB"
  server={filepondServer}
  onprocessfile={(error, file) => {
    // Handle successful upload
  }}
/>
```

FilePond provides:

1. Drag-and-drop file uploads
2. File type validation
3. File size validation
4. Image previews
5. Progress indicators

## Server Configuration

The FilePond server configuration defines how files are processed:

```javascript
const filepondServer = {
  url: process.env.FILEPOND_API_URL,
  process: function(fieldName, file, metadata, load, error, progress, abort) {
    // 1. Get a signed URL from Google Cloud Storage
    axios.post(`${process.env.FILEPOND_API_URL}${process.env.FILEPOND_API_ENDPOINT}`, {
      type: file.type,
      name: file.name,
      size: file.size,
    })
    .then(function (response) {
      const serverId = response.data.serverId;
      
      // 2. Upload the file directly to Google Cloud Storage
      axios.put(`${response.data.signedUrl}`, file, {
        headers: { 'content-type': file.type }
      })
      .then(() => {
        // 3. Return the server ID to FilePond
        load(serverId);
      })
      .catch((e) => {
        error();
      });
    })
    .catch((e) => {
      error();
    });
    
    // Return abort function
    return {
      abort: () => {
        // Cancel ongoing requests
        abort();
      }
    };
  },
  fetch: process.env.FILEPOND_API_ENDPOINT,
  revert: process.env.FILEPOND_API_ENDPOINT
};
```

## Micro Service Endpoint

The file upload endpoint is implemented as a Micro service:

```javascript
// node-routes/filepondRoute.js
const { json } = require('micro');
const cors = require('micro-cors')();
const { Storage } = require('@google-cloud/storage');

// Initialize Google Cloud Storage
const storage = new Storage();
const bucket = storage.bucket(process.env.GCS_BUCKET_NAME);

module.exports = cors(async (req, res) => {
  try {
    // Handle POST request for getting signed URL
    if (req.method === 'POST') {
      const body = await json(req);
      
      // Generate a unique filename
      const filename = generateUniqueFilename(body.name);
      
      // Create a signed URL for uploading
      const [signedUrl] = await bucket.file(filename).getSignedUrl({
        action: 'write',
        expires: Date.now() + 15 * 60 * 1000, // 15 minutes
        contentType: body.type,
      });
      
      // Generate an encrypted server ID
      const serverId = encryptFileId(filename);
      
      return {
        signedUrl,
        serverId,
      };
    }
    
    // Handle GET request for downloading files
    if (req.method === 'GET') {
      const fileId = req.url.split('/').pop();
      const filename = decryptFileId(fileId);
      
      // Create a signed URL for downloading
      const [signedUrl] = await bucket.file(filename).getSignedUrl({
        action: 'read',
        expires: Date.now() + 15 * 60 * 1000, // 15 minutes
      });
      
      // Redirect to the signed URL
      res.writeHead(302, { Location: signedUrl });
      res.end();
      return;
    }
    
    // Handle DELETE request for removing files
    if (req.method === 'DELETE') {
      const fileId = req.url.split('/').pop();
      const filename = decryptFileId(fileId);
      
      // Delete the file from Google Cloud Storage
      await bucket.file(filename).delete();
      
      return { success: true };
    }
    
    return { error: 'Method not allowed' };
  } catch (error) {
    console.error(error);
    return { error: 'Internal server error' };
  }
});
```

## File Encryption and Security

File IDs are encrypted to prevent direct access:

```javascript
const CryptoJS = require('crypto-js');

const SALT = process.env.SALT || ')6Dc1UP*S9Night-Age-Doll-Famous-8as81*@()#@';

// Encrypt a file ID
const encryptFileId = (filename) => {
  return CryptoJS.AES.encrypt(filename, SALT).toString();
};

// Decrypt a file ID
const decryptFileId = (fileId) => {
  return CryptoJS.AES.decrypt(fileId, SALT).toString(CryptoJS.enc.Utf8);
};

// Get filename from file ID
const getFilenameFromFileId = (fileId) => {
  const filename = _.last(decryptFileId(fileId).split('/'));
  return filename;
};
```

## File Types and Validation

The application supports various file types with validation:

| File Type | Accepted Formats | Size Limit | Purpose |
|-----------|------------------|------------|---------|
| Whitepaper | PDF, ZIP | 500MB | Project documentation |
| Presentation | PDF, ZIP | 500MB | Project presentation |
| Student ID | JPG, PNG, PDF | 10MB | Student verification |
| Transcript | PDF | 10MB | Academic verification |

Validation is performed on both client and server:

1. **Client-side**: FilePond plugins validate file type and size
2. **Server-side**: Additional validation before storage

## File Storage in Google Cloud

Files are stored in Google Cloud Storage with:

1. **Unique Filenames**: Generated to prevent collisions
2. **Content Type**: Preserved for proper serving
3. **Access Control**: Private by default, accessed via signed URLs
4. **Expiration**: Signed URLs expire after a set time

## File References in Database

File references are stored in the database:

```javascript
// Example of storing a file reference
const fileReference = {
  fileId: encryptedFileId,
  originalName: file.name,
  contentType: file.type,
  size: file.size,
  uploadedAt: new Date(),
};

// Add to the application record
application.whitepaperFileId = fileReference.fileId;
```

## File Retrieval

Files are retrieved using signed URLs:

```jsx
// In a component:
<a 
  target="_blank" 
  href={`${process.env.FILEPOND_API_URL}${process.env.FILEPOND_API_ENDPOINT}${fileId}`}
>
  {getFilenameFromFileId(fileId)}
</a>
```

When a user clicks the link:

1. Request goes to the file endpoint
2. File ID is decrypted to get the filename
3. A signed URL is generated for the file
4. User is redirected to the signed URL
5. File is downloaded or displayed

## Multiple File Uploads

For components that need multiple file uploads:

```jsx
// For student records with multiple files
{studentRecords.map((student, index) => (
  <FormSection key={index}>
    <h3>Student {index + 1}</h3>
    
    {/* Student ID Front */}
    <FormField>
      <label>Student ID (Front)</label>
      <FilePond
        ref={ref => this.pondRefs.studentCardFronts[index] = ref}
        allowMultiple={false}
        acceptedFileTypes="image/jpeg, image/png, application/pdf"
        maxFileSize="10MB"
        server={filepondServer}
        onprocessfile={(error, file) => {
          this.onFilepondChange(file, {
            name: "studentCardFrontFileId",
            section: "studentRecords",
            studentIndex: index
          });
        }}
      />
    </FormField>
    
    {/* Student ID Back */}
    <FormField>
      <label>Student ID (Back)</label>
      <FilePond
        ref={ref => this.pondRefs.studentCardBacks[index] = ref}
        allowMultiple={false}
        acceptedFileTypes="image/jpeg, image/png, application/pdf"
        maxFileSize="10MB"
        server={filepondServer}
        onprocessfile={(error, file) => {
          this.onFilepondChange(file, {
            name: "studentCardBackFileId",
            section: "studentRecords",
            studentIndex: index
          });
        }}
      />
    </FormField>
  </FormSection>
))}
```

## Best Practices

1. **Validate Files**: Always validate file types and sizes
2. **Secure Storage**: Use encrypted file IDs and signed URLs
3. **Error Handling**: Provide clear error messages for upload failures
4. **Progress Indicators**: Show upload progress to users
5. **File Previews**: When possible, show previews of uploaded files
6. **Cleanup**: Implement mechanisms to clean up unused files

