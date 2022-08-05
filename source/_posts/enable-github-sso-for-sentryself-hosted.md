---
title: Enable Github SSO for Sentry(self-hosted)
tags:
  - sentry
  - 记录
id: '12559'
categories:
  - - skills
date: 2022-06-12 21:40:39
---

## Getting Started

Once SSO be activated,

All existing users will recevie a email to link their account before they are able to continue using Sentry（**This will happen automatically once SSO is successfully configured**）.<!--more-->

Every member who creates a new account via SSO will be given global organization access with a member role.

### Steps

To enable Github SSO for Sentry , both Github and Sentry need to do some configuration.

#### Steps in Github

Create a new Github App :

*   Create Github App: [https://docs.github.com/en/developers/apps/building-github-apps/creating-a-github-app](https://docs.github.com/en/developers/apps/building-github-apps/creating-a-github-app)
    
*   Github App requires permissions and information to be filled in:[https://develop.sentry.dev/self-hosted/sso/#create-github-app-for-sso--integration](https://develop.sentry.dev/self-hosted/sso/#create-github-app-for-sso--integration)
    
*   Insatll Github App: [https://docs.github.com/en/developers/apps/managing-github-apps/installing-github-apps#installing-your-private-github-app-on-your-repository](https://docs.github.com/en/developers/apps/managing-github-apps/installing-github-apps#installing-your-private-github-app-on-your-repository)
    
*   After installation，you need save the following information:
    
    ```
    App ID
    GitHub App name
    Webhook secret
    Client ID
    Client Secret
    private-key
    ```
    

#### Steps in Sentry

*   Update configuration with Github App infomation

in `sentry/config.yaml`

```yaml
github-app.id: <App ID>
github-app.name: '<GitHub App name>'
github-app.webhook-secret: '<Webhook secret>' # Use only if configured in GitHub
github-app.client-id: '<Client ID>'
github-app.client-secret: '<Client secret>'
github-login.client-secre: '<Client secret>'
github-login.client-id: '<Client secret>'
github-app.private-key: 
  -----BEGIN RSA PRIVATE KEY-----
  privatekeyprivatekeyprivatekeyprivatekey
  privatekeyprivatekeyprivatekeyprivatekey
  privatekeyprivatekeyprivatekeyprivatekey
  privatekeyprivatekeyprivatekeyprivatekey
  privatekeyprivatekeyprivatekeyprivatekey
  -----END RSA PRIVATE KEY-----
```

*   restart
    
    ```bash
    docker-compose restart
    ```