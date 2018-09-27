###CloudServer

**Zenko CloudServer, an open-source Node.js implementation of the Amazon S3 protocol on the front-end and backend storage capabilities to multiple clouds, including Azure and Google**


*SCALITY_ACCESS_KEY_ID and SCALITY_SECRET_ACCESS_KEY*

These variables specify authentication credentials for an account named "CustomAccount".

You can set credentials for many accounts by editing conf/authdata.json (see below for further info), but if you just want to specify one set of your own, you can use these environment variables.
The default access key is accessKey1, with the secret key verySecretKey1

  environment:
    - SCALITY_ACCESS_KEY_ID=accessKey1
    - SCALITY_SECRET_ACCESS_KEY=verySecretKey1

**In production with Docker hosted CloudServer

In production, we expect that data will be persistent, that you will use the multiple backends capabilities of Zenko CloudServer, and that you will have a custom endpoint for your local storage, and custom credentials for your local storage:**

  volumes:
    - s3data:/usr/src/app/localData
    - s3data:/usr/src/app/localMetadata
volumes:
  s3data:
