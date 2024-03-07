# Salesforce on Roku

This is a proof-of-concept for a native Roku smart TV application that allows users to view and automatically refresh Salesforce Dashboards.

Use cases could include common scenarios where a dashboard is shown on a TV screen in an office, such as:

- Sales metrics in a sales office
- Performance dashboards and SLAs in a call center
- Event Monitoring dashboards in an IT department

There are no external services used - your Roku device directly interacts with Salesforce APIs, and relies on Salesforce infrastructure to generate the dashboard images that are displayed.

## Requirements

Please note this app will not work unless _all_ of the requirements below are met:

- Org must have CRM Analytics (fka. Tableau CRM, fka. Einstein Analytics, fka. Wave) enabled and configured, even if you only intend to use this app with Lightning Dashboards
- User account used to connect this app must have the appropriate licenses and permissions to view and download CRM Analytics Dashboards and Lightning Dashboards
- [CRM Analytics for Slack](https://help.salesforce.com/s/articleView?id=sf.crm_analytics_slack_app_intro.htm&type=5) must licensed and enabled. This is the mechanism that generates the screenshots of the dashboards we display in the app.

If in doubt, please ensure you can utilize the [Dashboard Download API](https://developer.salesforce.com/docs/atlas.en-us.api_analytics.meta/api_analytics/analytics_api_download_example_crma_dashboard_pdf.htm) successfully as the logged in user.

## User Guide

### Login

On app launch, you are prompted to log in. This login flow utilizes the [OAuth 2.0 Device Flow](https://help.salesforce.com/s/articleView?id=sf.remoteaccess_oauth_device_flow.htm&type=5) so users do not need to enter their passwords on the TV itself using the remote control, and to support SSO options commonly used in enterprise environments.

Scan the QR code which will take you to the pre-filled login initialization page, or simply browse to the URL manually and follow the instructions on screen.

![image](https://github.com/nzpei/roku-sfdc-poc/assets/3498834/7bb5238e-9ea8-4798-b040-010b42ebec9a)

To connect with Sandbox environments or to specify a custom My Domain, press the `Options (*)` button on your remote and select "Sandbox" or "My Domain" respectively.

![image](https://github.com/nzpei/roku-sfdc-poc/assets/3498834/fb4454ba-bd5c-494e-9e6b-d0751936739d)

For My Domain domains, only enter the portion before ".my.salesforce.com", for example, if your domain is "acme.my.salesforce.com" only enter in "acme".

![image](https://github.com/nzpei/roku-sfdc-poc/assets/3498834/2527179b-7708-4f9b-8f90-551679069644)

### Dashboard List

Browse and select the dashboard you wish to show. Note this screen fetches all "Recently Viewed" LEX and CRMA dashboards on load, so may take a while if you have a lot of assets in your org. Use the checkboxes on the left to hide and show LEX or CRMA dashboards respectively.

![image](https://github.com/nzpei/roku-sfdc-poc/assets/3498834/d3e99e30-94b4-406b-8510-b05c3766b76b)

### Dashboard View

By default, a dashboard opens in full-screen mode, zoomed to fill the screen.
![image](https://github.com/nzpei/roku-sfdc-poc/assets/3498834/6e97df9a-f69f-408d-8e4b-b53a2f91aec6)

In an ideal world, you should adapt and design your dashboard accordingly to optimize it for the screen size and resolution of your TV. A bit of trial and error may be required to get the size just right.

![image](https://github.com/nzpei/roku-sfdc-poc/assets/3498834/5911f302-1110-4f1c-8148-76e5c82b228a)

To focus on a particular part of the dashboard, if you wish to scroll up or down, use the arrow keys on the remote. The scrolled position is remembered even if the dashboard refreshes.

Alternatively, you can press the `OK` button to toggle between a zoomed-out view (whole dashboard displays on the screen with black bars on the side), and the default view.
![image](https://github.com/nzpei/roku-sfdc-poc/assets/3498834/4facfefc-c560-48d8-b80c-5fe907a601e2)

To navigate back to the dashboard selection screen, press the `Back` button.

### Automatically Refresh Dashboard

To periodically reload the dashboard, press the `*` button to toggle the options menu. Choose your desired refresh frequency with the `OK` button and then `Back` to exit.

![image](https://github.com/nzpei/roku-sfdc-poc/assets/3498834/f04c4789-bfaf-416a-a10c-f694906a7f3f)

The dashboard will automatically refresh based upon the refresh time specified. For LEX dashboards, an API call is made to explicitly "refresh" the dashboard prior downloading and displaying the screenshot. This may take a while if the dashboard itself is slow to refresh.

### Log Out

By default the app will remember your login even after closing/reopening the app or turning off your TV by persisting and using the [OAuth 2.0 Refresh Token Flow](https://help.salesforce.com/s/articleView?id=sf.remoteaccess_oauth_refresh_token_flow.htm&type=5)

To log out, press the `Back` button from the dashboard selection screen.

![image](https://github.com/nzpei/roku-sfdc-poc/assets/3498834/563d9959-4ef9-45af-88bd-cdc799615c92)

## Limitations

1. This app would likely fail certification for publishing in the Roku Store, due to the use of an off-device login flow. See [here](https://developer.roku.com/en-ca/docs/developer-program/authentication/authentication-and-linking.md) and [here](https://community.roku.com/t5/Roku-Developer-Program/Question-about-on-device-authentication/td-p/617264) . I suspect this is due to Roku's desire to have greater control over subscriptions - even for free apps - however this creates a much weaker security environment by requiring 3rd party apps to capture a user's password (as well as encouraging weaker, "easier to type using a TV remote" passwords being used). It would also be impossible to implement for orgs that mandate an SSO-based login provider (e.g. Okta), as Roku does not have any browser/webview components available on-platform. As a consequence, I have no intention of commercializing or officially releasing this app.
2. As the Roku device does not support any form of web browser, the only option to display dashboards is in the form of an image. To generate the screenshot/image, browser automation tools could be used and hosted somewhere - however this would require credentials for a user's org to be passed to an external service. In the interests of security, I wanted to avoid any scenario where a service I run (e.g. on AWS/Heroku) has access to access/refresh tokens for end users. The compromise however means that additional Salesforce feature licenses (CRM Analytics, and CRM Analytics for Slack) are prerequisites.

## Known Issue (Salesforce Bug)
(A workaround has been implemented in this code base due to the bug below, so users of this app will not experience this.)

I stumbled across a bizarre Salesforce bug with (what I believe is internally called "Pupparazzi" based on some error codes I've been able to make it spit out). 

#### Background:
Salesforce offers an "Analytics Download" API - a headless browser service running on Hyperforce infra, that grabs a screenshot of either a Lightning Dashboard, or a CRM Analytics Dashboard, and can return the result as a PNG file. 
This seems, in part, to be used for [integrating with Slack](https://help.salesforce.com/s/articleView?id=sf.crm_analytics_slack_app_intro.htm&type=5) but is also available via the REST API for developers to use.

It's documented in a couple of places:
- https://developer.salesforce.com/docs/atlas.en-us.api_analytics.meta/api_analytics/analytics_api_download_reference_download.htm
- https://developer.salesforce.com/docs/atlas.en-us.api_analytics.meta/api_analytics/analytics_api_download_example_le_dashboard_png.htm
- https://developer.salesforce.com/docs/atlas.en-us.api_analytics.meta/api_analytics/analytics_api_download_example_crma_dashboard_pdf.htm

#### Problem Statement
Once you've used an active session to grab one of the two types - LEX or CRMA - the other will no longer work until you get a new session. 

#### Steps to Repro:
1) Log in to an org that's has the appropriate feature licenses and config enabled, go through an OAuth flow to get an access_token, then do a GET request to /analytics/download/lightning-dashboard/<a_dashboard_id>.png => Result: Works, downloads a PNG for LEX dashboard

2) With the same session and access_token, try to do a GET request to /analytics/download/dashboard/<a_crma_dashboard_id>.png to try to grab a CRMA Dashboard => Result: FAILS, it will fail eventually with an HTTP 500 error code after a short period of time.

3) Redo your OAuth flow to get a brand new access_token, and do steps 1) and 2) in reverse. This time, the CRMA endpoint - accessed first - works, the LEX endpoint fails.

It seems whichever "type" of dashboard you attempt to download first using this API, does something​ in the backend that makes the other type fail. I have been able to consistently reproduce this behavior. Note: although not documented fully, it seems that instead of passing the typical "Authorization: Bearer <access_token>" header, you need to pass the sid via a cookie in order to access these endpoints. 

## Acknowledgements

Portions of the code used for this app, including most of the generic utility/library functions, overall app bootstrapping/loading architecture, and some dev tooling, has been derived from the [Playlet](https://github.com/iBicha/playlet) app by [Brahim Hadriche](https://github.com/iBicha).
