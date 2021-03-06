# PassCore: A self-service password change utility for Active Directory

![Passcore Logo](https://github.com/unosquare/passcore/raw/master/src/Unosquare.PassCore.Web/ClientApp/assets/images/passcore-logo.png)

[![Build status](https://ci.appveyor.com/api/projects/status/76nxqw893mk7xfb9/branch/master?svg=true)](https://ci.appveyor.com/project/geoperez/passcore/branch/master)
[![Github All Releases](https://img.shields.io/github/downloads/unosquare/passcore/total.svg)](https://github.com/unosquare/passcore/releases)
[![Analytics](https://ga-beacon.appspot.com/UA-8535255-2/unosquare/passcore/)](https://github.com/igrigorik/ga-beacon)

*:star:Please star this project if you find it useful!*

- [PassCore: A self-service password change utility for Active Directory](#passcore-a-self-service-password-change-utility-for-active-directory)
  - [Overview](#overview)
    - [Features](#features)
  - [Installation on IIS](#installation-on-iis)
  - [Customization and Configuration](#customization-and-configuration)
    - [Running as a sub application](#running-as-a-sub-application)
  - [Troubleshooting](#troubleshooting)
    - [LDAP Support (under review)](#ldap-support-under-review)
  - [License](#license)

## Overview

PassCore is a very simple 1-page web application written in [C#](https://docs.microsoft.com/en-us/dotnet/csharp/), using [ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/getting-started?view=aspnetcore-2.0) , [Angular Material](https://material.angular.io/), [Typescript](http://www.typescriptlang.org/), and [Microsoft Directory Services](https://docs.microsoft.com/en-us/dotnet/api/system.directoryservices). It allows users to change their Active Directory password on their own, provided the user is not disabled.

PassCore does not require any configuration, as it obtains the principal context from the current domain. I wrote this because a number of people have requested several features that the [original version](http://unopasscore.codeplex.com/) did not have. The original version of this tool was downloaded around 8000 times in 2.5 years. My hope is that the new version continues to be just as popular. There really is no free alternative out there (that I know of) so hopefully this saves someone else some time and money.

### Features

PassCore has the following features:

- Easily localizable (i.e. you can customize all of the strings in the UI -- see the section on Customization)
- Supports [reCAPTCHA](https://www.google.com/recaptcha/intro/index.html)
- Has a built-in password meter
- Responsive design that works on mobiles, tablets, and desktops.

<img align="center" src="https://github.com/unosquare/passcore/raw/master/preview.png"></img>

## Installation on IIS

1. Ensure the server running IIS is domain-joined. To determine if the computer is domain-joined:
    - Go to the *Start* menu, right click on *Computer*, then select *Properties*
    - Make sure the *Domain* field contains the correct setting.
1. You need a Passcore copy to continue. We recommed to download the latest binary release of [PassCore](https://github.com/unosquare/passcore/releases/download/3.2.1/PassCore321.zip).
1. **NOTE:** Before extracting the contents of the file, please right click on it, select Properties and make sure the file is Unblocked (Click on the Unblock button at the bottom of the dialog if it is available). Then, extract the contents of the zip file to the directory where you will be serving the website from.
    - If you download the source code you need to run the following command via an Command Prompt. Make sure you start the Command Prompt with the Administrator option.
    - `dotnet publish --configuration Release --runtime win-x64 --output "<path>"`
    - The `<path>` is the directory where you will be serving the website from.
1. Install the [.NET Core Windows Server Hosting bundle](https://docs.microsoft.com/en-us/aspnet/core/publishing/iis?tabs=aspnetcore2x#install-the-net-core-windows-server-hosting-bundle). You will need at least [.NET Core Runtime 2.1.0 / SDK 2.1.300](https://www.microsoft.com/net/download/windows).
1. Go to your *IIS Manager*, Right click on *Application Pools* and select *Add Application Pool*.
1. A dialog appears. Under Name enter **PassCore Application Pool**, Under .NET CLR Version select **No Managed Code** and finally, under Managed pipeline mode select **Integrated**. Click OK after all fields have been set.
1. Now, right click on the application pool you just created in the previous step and select *Advanced Settings ...*. Change the *Start Mode* to **AlwaysRunning**, and the *Idle Time-out (minutes)* to **0**. Click on *OK*. This will ensure PassCore stays responsive even after long periods of inactivity.
1. Back on your *IIS Manager*, right click on *Sites* and select *Add Website*
1. A dialog appears. Under *Site name*, enter **PassCore Website**. Under *Application pool* click on *Select* and ensure you select **PassCore Application Pool**. Under *Physical path*, click on the ellispsis *(...)*, navigate to the folder where you extracted PassCore.
    - **Important:** Make sure the Physical path points to the *parent* folder which is the one containing the files, *logs* and *wwwroot* folders.
    - **NOTE:** If the folder `logs` is not there you can created. To enable the logs you need to change `stdoutLogEnabled` to `true` in the `web.config` file.
1. Under the *Binding section* of the same dialog, configure the *Type* to be **https**, set *IP Address* to **All Unassigned**, the *Port* to **443** and the *Host name* to something like **password.yourdomain.com**. Under *SSL Certificate* select a certificate that matches the Host name you provided above. If you don't know how to install a certificate, please refer to [SSL Certificate Install on IIS 8](https://www.digicert.com/ssl-certificate-installation-microsoft-iis-8.htm) or [SSL Certificate Install on IIS 10](https://www.digicert.com/csr-creation-ssl-installation-iis-10.htm) , in order to install a proper certificate.
    - **Important:** Do not serve this website without an SSL certificate because requests and responses will be transmitted in cleartext and an attacker could easily retrieve these messages and collect usernames and passwords.
1. Click *OK* and navigate to `https://password.yourdomain.com` (the host name you previously set). If all is set then you should be able to see the PassCore tool show up in your browser.

**NOTE:** If you have a previous version, you **can not** use the same `appsettings.json` file. Please update your settings manually editing the new file.

## Customization and Configuration

All server-side settings and client-side settings are stored in the `/appsettings.json` file.
The most relevant configuration entries are shown below. Make sure you make your changes to the `appsettings.json` file using a regular text editor like [Visual Studio Code](https://code.visualstudio.com)

- To enable reCAPTCHA
  1. reCaptcha is enable in testing mode, please change the keys to use it with your application
  2. Find the `RecaptchaPrivateKey` entry and enter your private key within double quotes (`"`)
  3. Find the `SiteKey` entry and enter your Site Key within double quotes (`"`)
- To change the language of the reCAPTCHA widget
  - Find the `LanguageCode` entry and enter [one of the options listed here](https://developers.google.com/recaptcha/docs/language). By default this is set to `en`
- To enable the password meter
  - Find the `ShowPasswordMeter` entry and set it to `true` (without quotes)
- To disable the password meter
  - Find the `ShowPasswordMeter` entry and set it to `false` (without quotes)
- To enable restricted group checking
  1. Find the `CheckRestrictedAdGroups` entry and set it to `true` (without quotes)
  2. Find the `RestrictedADGroups` entry and add any groups that are sensitive.  Accounts in these groups (directly or inherited) will not be able to change their password.
- Find the `DefaultDomain` entry and set it to your default Active Directory domain. This should eliminate confusion about using e-mail domains / internal domain names.
- To provide an optional paramerter to the URL to set the username text box automatically
  1. `http://mypasscore.com/?userName=someusername`
  2. This helps the user incase they forgot thier username and, also comes in handy when sending a link to the application or having it embeded into another application were the user is all ready signed in.
- The rest of the configuration entries are all pretty much all UI strings. Change them to localize, or to brand this utility, to meet your needs.

### Running as a sub application

To run as a sub application you need to modify the `base href="/"` value in the `wwwroot/index.html` file to be the base url for PassCore. For example you might have PassCore setup at /PassCore so you would put

```html
<base href="/PassCore/" />
```

## Troubleshooting

- At first run if you find an error (e.g. **HTTP Error 502.5**) first ensure you have installed [.NET Core Runtime 2.1.0 / SDK 2.1.300](https://www.microsoft.com/net/download/windows), or better.
- If you find an [HTTP Error 500](https://stackoverflow.com/questions/45415832/http-error-500-19-in-iis-10-and-visual-studio-2017) you can try
  1. Press Win Key+R to Open Run Window
  1. in the Run Window, enter "OptionalFeatures.exe"
  1. in the features window, Click: "Internet Information Services"
  1. Click: "World Wide Web Services"
  1. Click: "Application Development Features"
  1. Check the features.
- If you / your user's current password never seems to be accepted for reset; the affected person may need to use a domain-connected PC to login and reset their password on it first. Updated group policy settings could be blocking user changes, until a local login is completed.

### LDAP Support (under review)

- If your users are having trouble changing passwords as in issues #8 or #9 : try configuring the section `PasswordChangeOptions` in the `/appsettings.json` file. Here are some guidelines:
  1. Ensure `UseAutomaticContext` is set to `false`
  1. Ensure `LdapUsername` is set to an AD user with enough permissions to reset user passwords
  1. Ensure `LdapPassword` is set to the correct password for the admin user mentioned above
  1. User @gadams65 suggests the following: Use the FQDN of your LDAP host. Enter the LDAP username without any other prefix or suffix such as `domain\\` or `@domain`. Only the username.

## License

PassCore is open source software and MIT licensed. Please star this project if you like it.
