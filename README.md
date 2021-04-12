# Generacion de peticion de Dataflow-GCS
## Generar Token
En el cli corres esta linea:
```console
gcloud auth print-identity-token
```
## Generar la peticion
Una vez generado el token, sigue hacer la peticion, en este caso, necesitas hacer una peticion POST
Este es un ejemplo que nos muestra Google:
```console
curl https://REGION-PROJECT_ID.cloudfunctions.net/FUNCTION_NAME   -H "Authorization: bearer $(gcloud auth print-identity-token)"
```
Con ```Python``` es:
```python
import requests

url = "https://us-central1-raeltest.cloudfunctions.net/segmentador-test-gcs"

payload = {
    "query": "SELECT * FROM `raeltest.bi.cat_bodegasporciudad`",
    "filename": "cat_bodegasporciudad.csv",
    "unique_id": "1460"
}
headers = {
    "Content-Type": "application/json",
    "Authorization": "bearer [token]"
}

response = requests.request("POST", url, json=payload, headers=headers)

print(response.text)
```

Las pruebas es posibles hacerla con Insomnia:
[link to Insomnia!](https://insomnia.rest/download)

## Preview:

## Insomnia Timeline:
```
* Preparing request to https://us-central1-raeltest.cloudfunctions.net/segmentador-test-gcs
* Current time is 2021-04-12T14:52:42.052Z
* Using libcurl/7.73.0 OpenSSL/1.1.1j zlib/1.2.11 brotli/1.0.9 zstd/1.4.9 libidn2/2.1.1 libssh2/1.9.0 nghttp2/1.41.0
* Using default HTTP version
* Disable timeout
* Enable automatic URL encoding
* Enable SSL validation
* Enable cookie sending with jar of 0 cookies
* Too old connection (328 seconds), disconnect it
* Connection 0 seems to be dead!
* Closing connection 0
* TLSv1.3 (OUT), TLS alert, close notify (256):
* Hostname in DNS cache was stale, zapped
*   Trying 216.239.36.54:443...
* Connected to us-central1-raeltest.cloudfunctions.net (216.239.36.54) port 443 (#1)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*  CAfile: /var/folders/d1/6kktx955279bwgqrs27bvt840000gn/T/insomnia_2021.2.2/ca-certs.pem
*  CApath: none
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
* TLSv1.3 (IN), TLS handshake, Certificate (11):
* TLSv1.3 (IN), TLS handshake, CERT verify (15):
* TLSv1.3 (IN), TLS handshake, Finished (20):
* TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.3 (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384
* ALPN, server accepted to use h2
* Server certificate:
*  subject: C=US; ST=California; L=Mountain View; O=Google LLC; CN=misc.google.com
*  start date: Mar 16 19:28:41 2021 GMT
*  expire date: Jun  8 19:28:40 2021 GMT
*  subjectAltName: host "us-central1-raeltest.cloudfunctions.net" matched cert's "*.cloudfunctions.net"
*  issuer: C=US; O=Google Trust Services; CN=GTS CA 1O1
*  SSL certificate verify ok.
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* Using Stream ID: 1 (easy handle 0x7fc9cb320600)

> POST /segmentador-test-gcs HTTP/2
> Host: us-central1-raeltest.cloudfunctions.net
> user-agent: insomnia/2021.2.2
> content-type: application/json
> authorization: bearer eyJhbGciOiJSUzI1NiIsImtpZCI6ImUxYWMzOWI2Y2NlZGEzM2NjOGNhNDNlOWNiYzE0ZjY2ZmFiODVhNGMiLCJ0eXAiOiJKV1QifQ.eyJpc3MiOiJhY2NvdW50cy5nb29nbGUuY29tIiwiYXpwIjoiNjE4MTA0NzA4MDU0LTlyOXMxYzRhbGczNmVybGl1Y2hvOXQ1Mm4zMm42ZGdxLmFwcHMuZ29vZ2xldXNlcmNvbnRlbnQuY29tIiwiYXVkIjoiNjE4MTA0NzA4MDU0LTlyOXMxYzRhbGczNmVybGl1Y2hvOXQ1Mm4zMm42ZGdxLmFwcHMuZ29vZ2xldXNlcmNvbnRlbnQuY29tIiwic3ViIjoiMTE0MjU1MjUwMTg3MTUzMDg2ODM1IiwiZW1haWwiOiJyYWVsY29ycmFsZXNAZ21haWwuY29tIiwiZW1haWxfdmVyaWZpZWQiOnRydWUsImF0X2hhc2giOiJFT3M0Q0lYNG9nMFRRR2hVaUtPU0xnIiwiaWF0IjoxNjE4MjM4NjAzLCJleHAiOjE2MTgyNDIyMDMsImp0aSI6IjljZGU3NjZlYmFjNmIxMDIyNGYzZDdhYTUxY2ZlN2UwZDZlMjg4NDIifQ.VfqLJQjZP36QkCgnBjBe9EywP-3A8mq1xfQg7Vsw3A_25zLFzjcXtJ6B-gCARo5nGzaFWWI6lmZhQRyEQRB8iZnV1ou49T-3swFu4k-JXv2zTrlNYAAC8Yws-D1SRm9IQbc2O0WpW7P95pGMtC85m3hhcqIdyo5XkNH2d5HGvr2lVf0lVG4cfFYfkHS_CevNo37-mLPWHG-CpLr40TbTxWtuJVJs_bKMhdhIc7v7CpGN8DGEn6lfccIQH_kP55TNPFTh9VN7u9rsnRdEpYP_PLEyQgiUopghgpMzMaGNgBuzYNAuUWGDz9w4MpkY-iCAqI0KxAoklaLENpmszr0w1Q
> accept: */*
> content-length: 127

| {
| 	"query": "SELECT * FROM `raeltest.bi.cat_bodegasporciudad`",
| 	"filename": "cat_bodegasporciudad.csv",
| 	"unique_id": "1460"
| }

* We are completely uploaded and fine
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* old SSL session ID is stale, removing
* Connection state changed (MAX_CONCURRENT_STREAMS == 100)!

< HTTP/2 200 
< content-type: application/json; charset=utf-8
< etag: W/"75-Qjc4nAyI/4CfjJjdr64gTLF6i/M"
< function-execution-id: n0y5yilviy6w
< x-powered-by: Express
< x-cloud-trace-context: 41672a18c310f2aee4bd5afc1793c8e1;o=1
< date: Mon, 12 Apr 2021 14:54:49 GMT
< server: Google Frontend
< content-length: 117
< alt-svc: h3-29=":443"; ma=2592000,h3-T051=":443"; ma=2592000,h3-Q050=":443"; ma=2592000,h3-Q046=":443"; ma=2592000,h3-Q043=":443"; ma=2592000,quic=":443"; ma=2592000; v="46,43"


* Received 117 B chunk
* Connection #1 to host us-central1-raeltest.cloudfunctions.net left intact
```
