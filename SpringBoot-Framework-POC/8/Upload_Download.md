# RESTful Upload & Download

### Task complete

1. Upload a file with POST request
2. Download a file with GET request

<br>
## 1. Maven Dependencies

```xml
<dependency>
	<groupId>javax.mail</groupId>
	<artifactId>mail</artifactId>
	<version>1.4.7</version>
</dependency>
```

<br>
## 2. Upload File with a POST call

Upload a file with a POST call that consumes multipart/form-data.

**Camel Route**

- Define an entry REST endpoint to receive the POST call
- Process the request **body** for saving the file
- `.to()` the path you save the file

```java
from("rest:post:upload?consumes=multipart/form-data")
	.process(new Processor() {
		@Override
		public void process(Exchange ex) throws Exception {
			InputStream is = ex.getIn().getBody(InputStream.class);
			MimeBodyPart mimeMessage = new MimeBodyPart(is);
			DataHandler dh = mimeMessage.getDataHandler();
			ex.getIn().setBody(dh.getInputStream());
			ex.getIn().setHeader(ex.FILE_NAME, dh.getName());
		}
	}).to("file:/yourPath");
```

**Postman Call**

For testing by a Postman POST call, set the body to form-data, with the key is ‘file’, and place the file to be uploaded to the value.

<br>
## 3. Download File with a GET call

- Get the requested filename from the path
- Set the output header **Content type** to “application/octet-stream”
- Set the output header **Content-Disposition** with the output filename
                                   *(Content-Disposition is String as dk why can’t import correct one)*

```java
from("rest:get:download/{filename}")
	.process(ex -> {
		String fileName = ex.getIn().getHeader("fileName", String.class);
		String filePath = "/your_path/" + fileName;
		File file = new File(filePath);
		ex.getOut().setBody(file);
		ex.getOut().setHeader(ex.CONTENT_TYPE, MediaType.APPLICATION_OCTET_STREAM);
		ex.getOut().setHeader("Content-Disposition", "attachment; filename=\"" + fileName + "\"");
	});
```