# **__Generate an API key from Cortex__**

So that we can integrate these 2 systems we need an API key from Cortex. The best way of doing this is to login with your orgadmin account (not the superadmin from the parent org though). Create a new integration user. Give this user read, analyze permission.

![](../images/cortex-api-1.png)

Once created select Create API key and then reveal. Make a note of this key as you need it for TheHive configuration.

![](../images/cortex-api-2.png)

Now after getting the API-Key you have to store it at somewhere safe place to use it in Thehive server Configuration.

## Modify TheHive configuration file

Within TheHive’s application.conf (which ive place in /opt/thehive/conf) scroll down to the section labeled Cortex and make the following changes

### Shown in the Installation guide of TheHive

