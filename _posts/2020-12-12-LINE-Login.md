---
layout: post
title:  "LINE Login"
date:   2020-12-13 00:00:30 +0800
categories: LINE
---

### Create a Line channel

Create a channel &rArr; LINE Login &rArr; Fill out and Click **Create**

##### Access the user's email address

under **OpenID Connect**, click **Apply**.



### Design a Login button

 You can download [here](https://developers.line.biz/en/docs/line-login/login-button/).

### API handling

*Step 1* 

- [LINE-Login page](#line-login-page)

*Step 2* 

- [Get Access Token](#get-access-token)

*Step 3* 

- [Get user's profile](#get-users-profile)

#### LINE-Login page

Jump to the login page.

>  https://access.line.me/oauth2/v2.1/authorize

| Parameters      | Description                |
| --------------- | -------------------------- |
| `response_type` | `code`                     |
| `client_id`     | Channel ID                 |
| `redirect_uri`  | callback URL               |
| `state`         | random value (prevent XXS) |
| `scope`         | profile, email, openid     |

```javascript
// Example
function LineAuth() {
    var client_id = 'channel ID';
    var redirect_uri = 'https://example.com/callback';
    var state = 'abcde';
    var scope = 'openid%20profile%20email';
    var URL = 'https://access.line.me/oauth2/v2.1/authorize?'
        + `response_type=code`
        + `&client_id=${client_id}`
        + `&redirect_uri=${redirect_uri}`
        + `&state=${state}`
        + `&scope=${scope}`;
    window.location.href = URL;
}
```

Login succeed and redirect to `redirect_uri` which contains querystrings.

>  https://example.com/callback?code=abcd1234&state=abcde

Check the error message and compare if state is equal.

```javascript
function CheckQueryStr(){
    var res = {};
    var parameters = window.location.search.substring(1);
    res = JSON.parse('{"' + decodeURI(parameters).replace(/"/g, '\\"')
                     .replace(/&/g, '","').replace(/=/g, 	'":"') + '"}');
    'error' in res ? alert('error') :
    	res.state !== 'abcde' ? alert('state is not the same') :
    		PassAccesstoken()
}
```

#### Get Access Token

> https://api.line.me/oauth2/v2.1/token

| Parameters      | Description          |
| --------------- | -------------------- |
| `grant_type`    | `authorization_code` |
| `code`          | code                 |
| `redirect_uri`  | Callback URL         |
| `client_id`     | Channel ID           |
| `client_secret` | Channel secret       |

```c#
//Example
void Page_Load(Object sender, EventArgs e)
{
    string url = "https://api.line.me/oauth2/v2.1/token";
    string code = Request["code"];

    HttpWebRequest request = (HttpWebRequest)WebRequest.Create(url);
    request.Method = "POST";
    request.ContentType = "application/x-www-form-urlencoded";

    NameValueCollection postParams = System.Web.HttpUtility.ParseQueryString(string.Empty);
    postParams.Add("grant_type", "authorization_code");
    postParams.Add("code", code);
    postParams.Add("redirect_uri", "https://example.com/callback");
    postParams.Add("client_id", "xxxxxx");
    postParams.Add("client_secret", "xxxxxxx");

    byte[] bs = Encoding.ASCII.GetBytes(postParams.ToString());
    using (Stream reqStream = request.GetRequestStream())
    {
        reqStream.Write(bs, 0, bs.Length);
    }

    try
    {
        using (WebResponse response = request.GetResponse())
        {
            StreamReader sr = new StreamReader(response.GetResponseStream());
            TokenObject tokenObject = JsonConvert.DeserializeObject<TokenObject>(sr.ReadToEnd());
            sr.Close();

            string json_profile = JwtParser(tokenObject.id_token);

            LineProfile lineProfile = JsonConvert.DeserializeObject<LineProfile>(json_profile);
            Response.Write(lineProfile.sub);
        }
    }
    catch(Exception ex)
    {
        Response.Write(ex.Message);
    }

}
```

Response

```json
{
    "access_token": "eyJhbGciOiJIUzI1NiJ9.bJo9x...",
	"expires_in": 2592000,
	"id_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUz...",
	"refresh_token": "xxxxxxxxxxx",
	"scope": "openid profile",
	"token_type": "Bearer"
}
```

#### Get user's profile

Decode the id_token which is JWT.

```javascript
var profile = JSON.parse(atob(id_token.split('.')[1]));
```

```c#
public string JwtParser(string id_token)
{
    string str = id_token.Split('.')[1];
    int mod4 = str.Length % 4;
    if (mod4 > 0)
    {
    	str += new string('=', 4 - mod4);
    }

    byte[] data = Convert.FromBase64String(str);
    string base64Decoded = ASCIIEncoding.ASCII.GetString(data);

    return base64Decoded;
}
```

Use a LINE Login API endpoint.

> https://api.line.me/oauth2/v2.1/verify

| Parameters | Description |
| ---------- | ----------- |
| `id_token` | id_token    |

Use a LINE profile API.

> https://api.line.me/v2/profile

| **Authorization** | Description  |
| ----------------- | ------------ |
| `Bearer_token`    | access_token |



Response

```json
{
    "iss": "https://access.line.me",
    "sub": "user UID",
    "aud": "channel ID",
    "exp": 1504169092,
    "iat": 1504263657,
    "amr": [
        "pwd",
        "linesso",
        "lineqr"
    ],
    "name": "display name",
    "picture": "https://sample_line.me/aBcdefg123456",
    "email": "email address"
}
```