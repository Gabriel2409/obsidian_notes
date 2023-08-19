---
sr-due: 2023-07-04
sr-interval: 1
sr-ease: 228
reviewed: 2023-07-20
---

#snowflake

## File support REST API

- GET /api/files
- Takes a scoped URL (BUILD_SCOPED_FILE_URL) or a File URL (BUILD_STAGE_FILE_URL) but can not take a presigned url
- For scoped urls, only the user who generated the url can download the staged file
- For file urls, any role that has privileges on the underlying stage can access the file (USAGE for external and READ for internal)

```python
import requests

url = "https://<locator>.<region>.<provider>.snowflakecomputing.com/api/files/<db>/<schema>/<stage>/<path>/<to>/<file>"
headers = {
	"User-Agent": "reg-tests",
	"Accept": "*/*",
	"X-Snowflake-Authorization-Token_Type": "OAUTH",
	"Authorization": f"Bearer {token}"
}
response = requests.get(url, headers = headers,	allow_redirects = True)
print(response.status_code)
print(response.content)
```
