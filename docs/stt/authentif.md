---
sidebar_position: 5
---

# Authentication

## General

Typeless STT APIs uses API keys to authenticate requests. You have received or will receive an API key from us. This API key will be shared between all your users. You should use the API key to make requests to the API from your backend.

## Authentication flow

![Authentication](/img/auth.png)

**Figure:** Overview of advised authentication flow

## Advices

- **Do not use the API key in the frontend**:

  The API key is a secret and should not be exposed to the public. If you use the API key in the frontend, it will be visible to everyone and can be used to make requests on your behalf. Instead, use the API key in your backend and make requests to the API from there.

- **Reverse proxy**:

  If you use the API key in your backend, we advise you to use a reverse proxy (such as **Nginx**):

  - The reverse proxy will be the only one to know the API key.
  - It will forward the requests from the frontend to the API.
  - It will add the API key to the requests.
  - It will forward the responses from the API to the frontend.

```nginx
# Example of nginx configuration
server {
    listen 443 ssl;
    server_name <YOUR_DOMAIN>/<YOUR_EXTENSION_FOR_OUR_SERVICE>;

    ssl_certificate <PATH_TO_CERTIFICATE>;
    ssl_certificate_key <PATH_TO_CERTIFICATE_KEY>;

    location /real-time-transcription {
        rewrite ^/<YOUR_EXTENSION_FOR_OUR_SERVICE>(.*)$ $1 break;
        proxy_pass https://speech.typeless.ch;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
        proxy_set_header Authorization "Bearer <YOUR_TYPELESS_API_KEY>";
    }
}
```

- **Your authentification**:

  We advise you to use your own authentification system to authenticate your users before feeding any Typeless request to the reverse proxy.
