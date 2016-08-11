---
layout: post
title: Hosting Django sites on IIS
---

The following instructions outline the steps to get a Django 1.7/1.8/1.9 site running on IIS 7/8.

These instructions borrow heavily from <http://codesmartinc.com/2013/04/12/running-django-in-iis7iis8/>.  Unfortunately, those instructions are out of date for Django 1.7/1.8 and do not work.

Please note that wfastcgi.py will need to be copied into the project directory.  If you do not already have this file you can obtain it from the Python installation installed by the Web Platform installer (see original instructions references above for more info).

1. Copy your django project into c:\inetpub\wwwroot\<project name>

2. Copy the wfastcgi.py file referenced above into c:\inetpub\wwwroot\<project name>

3. Open IIS Manger

4. Select the root web server and verify icons for CGI and FastCGI Settings are present.

    If the icons are not present then you will need to enable the CGI feature.  

    1. Open Server Manager
    2. Click Add Roles and Features from the Manage menu
    3. Navigate through the Wizard until you get to the Server Roles section
    4.  Check the CGI box under Web Server (IIS), Web Server, Application Development
    5. Click Install
  
5.  Create a new website and set the physical path to c:\inetpub\wwwroot\<project name>

    If you are hosting multiple web sites on the sever and they need to share port 80 or 443 then you will need to set the host name for each site to avoid port conflicts. You may need to add to add an entry to the host file for the specified sub-domain to access it from the local server.

    If you want to access the site using <project_name>.<domain>.com you will need to enter that in full name for the Host name.  You will need to add an entry to the hosts file that maps <project_name>.<domain>.com to 127.0.0.1.  You may also need to add a entry to your DNS zone file to map it to the public IP address of the web server.
    
7.  Click Add Module Mapping (located in actions pane on the right side) and use the following settings:

    Request Path: *<br>
    Module FastCgiModule    
    Executable: C:\Python27\python.exe|C:\inetpub\wwwroot\<project_name>\wfastcgi.py
    
    Adjust the Python path accordingly for Python 3.
    
    If you are using a virtual environment you should specify the path to the python executable located in your virtual environment.  Here is an example:
    
    C:\virtual_environments\<project_name>\Scripts\python.exe|C:\inetpub\wwwroot\<project_name>\wfastcgi.py
    
8. Click Request Restrictions

9. Uncheck the option to invoke handler only if requested is mapped

10. Click OK to close the Request Restrictions dialog box

11. Click OK to close the Edit Module Mapping dialog box

12. Click YES when prompted to add an entry to the FastCGI Collection in IIS

13. Select the root server

14.  Open FastCGI Settings

15. Find the entry to the python path you entered in step 7 and open it

16. Open the Environment Variables collection property box (click the … button)
    
17.  Add the following environment variables:

    DJANGO_SETTINGS_MODULE: <project_name>.settings
    
    If you are using multiple setting files as recommended by Two Scoops of Django then your DJANGO_SETTINGS_MODULE variable may look like this: <project_name>.settings.production
    
    PYTHONPATH: c:\inetpub\wwwroot\<project_name>  
    WSGI_HANDLER: django.core.wsgi.get_wsgi_application()
    
    Additional Environment variables
    
    If you are storing secrets in environment variables (such as secret keys and database credentials) as recommended by Two Scoops of Django then you would add them here as well.  Please note that you’ll still need to set these additional environment variables at the console to run manage.py commands.    

18.  Click OK to close the Environment Variables Collection Editor dialog box.

19. Click OK to close the Edit FastCGI application dialog box.

20.  Browse to the site and enjoy.

###Serving Static Files

You must configure IIS to host the static files.

1.  Right click the web site and choose Add Virtual Directory

2.  Configure the following settings:

    Alias - This will be the name specified for STATIC_URL in your project’s settings file.  
    Physical Path - This will be name specified for STATIC_ROOT in your project’s setting file.

3. Click ok to close the Add Virtual Directory dialog box

4. Select the newly created virtual directory

5. Open Handler Mappings

6. Click View Ordered List (located in actions pane on the right side) 

7.  Select StaticFile and click Move Up until the entry is at the top of the list.  

    Click Yes on the Handler Mappings warning dialog box informing you that changing made at the parent level will no longer be inherited at this level.
    
###Serving Media Files

You must configure IIS to host the media files.

1.  Right click the web site and choose Add Virtual Directory

2.  Configure the following settings:

    Alias - This will be the name specified for MEDIA_URL in your project’s settings file.  
    Physical Path - This will be name specified for MEDIA_ROOT in your project’s setting file.
    
3. Click ok to close the Add Virtual Directory dialog box

4. Select the newly created virtual directory

5. Open Handler Mappings

6. Click View Ordered List (located in actions pane on the right side) 

7.  Select StaticFile and click Move Up until the entry is at the top of the list.  

    Click Yes on the Handler Mappings warning dialog box informing you that changing made at the parent level will no longer be inherited at this level.