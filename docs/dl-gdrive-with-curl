Download files from Google Drive on CLI
=======================================
Small files on Google Drive can be downloaded on the CLI. Every file has an ID. For example, the ID for:

```
https://drive.google.com/file/d/00-00-abcxYZIJK123/view
```

is

```
00-00-abcxYZIJK123
```

If the file is not too big, downloading is simple:


```bash
curl "https://drive.google.com/uc?export=download&id=${file_id}" -O myfile.zip
```

Downloading big files from Google Drive is a bit more involved. You need **BOTH** the cookie and confirm code from the cookie:

```bash
curl -sc /tmp/cookie "https://drive.google.com/uc?export=download&id=${file_id}"
cfm_code=$(awk '/_warning_/ {print $NF}' /tmp/cookie)
curl -Lb /tmp/cookie "https://drive.google.com/uc?export=download&confirm=${cfm_code}&id=${file_id}" -o myfile.zip
rm /tmp/cookie
```
