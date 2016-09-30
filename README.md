# minio-zipstream 

This is a fork from [kbbdy/zipstream](https://github.com/kbbdy/zipstream) for implementing multiple object download from S3 backend.

Simple python library for streaming ZIP files which are created dynamically, without using any temporary files.

Files stored in ZIP file are not compressed. Its intended to serve easily structured content in convenient way in web applications

- No temporary files, data is streamed directly from files
- Small memory usage, straming is realised using yield statement
- Archive structure is created on the fly
- Zip32 compatible files
- Zip64 support is planned in future
- Independent from python's standard lib implementation


## Examples:

### Example of creating zip file

```python
from zipstream import ZipStream
from minio import Minio
from minio.error import ResponseError

s3Client = Minio('s3.amazonaws.com',
                 access_key='ACCESS_KEY',
                 secret_key='SECRET_KEY')

zs = ZipStream()

# List all object paths in bucket that begin with my-prefixname.
objects = s3client.list_objects('mybucket', prefix='my-prefixname',
                                   recursive=True)
for obj in objects:
    zs.add_file(obj.object_name, obj.size, obj.last_modified)

# write result file
with file("attachment.zip","wb") as fout:
    for f in zs.stream():
        fout.write(f)
```

### Example of using zipstream as Django view

```python
from django.http import StreamingHttpResponse
from os.path import basename as get_object_base_name

def stream_as_zip(request, object_names):
    streamed_data_filename = "my_streamed_zip_file.zip"
    # large chunk size will improve speed, but increase memory usage
    stream = ZipStream(chunksize=32768)
    
    for object_name in object_names:
        # filename of first file in ZIP archive will be different than original
        stream.add_file(get_object_base_name(object_name))

    # streamed response
    response = StreamingHttpResponse(
        stream.stream(),
        content_type="application/zip")
    response['Content-Disposition'] =
        'attachment; filename="%s"' % streamed_data_filename
    return response
```
