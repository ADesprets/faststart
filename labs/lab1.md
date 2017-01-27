---
title: Lab 1 - Customising the API Connect Developer Portal
toc: true
sidebar: labs_sidebar
permalink: /europe/2017/lab1.html
summary: In this lab, you’ll get a chance to use the APIC command line interface for creating LoopBack applications, the intuitive Web-based user interface, and explore the various aspects associated with solution’s configuration of RESTful based services as well as their operation.
---

## Objective 
In the following lab, you will learn:

+ How to create a simple LoopBack application
+ How to create a Representational State Transfer (REST) API definition
+ How to test a REST API
+ How to publish an API to the Developer Portal

## Case Study used in this tutorial
In this tutorial, you will be starting from scratch to set up your development environment on your local machine, and then you will test the components of the toolkit by creating the default Notes LoopBack application. We will look at the created artifacts to get a better understanding of what is created for you. The notes application is a simple RESTful service that holds a set of "notes" in memory. In this lab we may be creating a new RESTful service and API, but in future labs you'll be creating APIs with existing services.

## Before you begin
For this lab, you will be starting with your local image and installing node.js and the developer toolkit.  After that, if you do not already have a Bluemix account, you will be creating one for you and enabling the API Connect Essentials service on your account. The instructions you will follow will vary by the host operating system you are using. The three host OS's supported by this are `Windows`, `Linux - Intel` and `Linux - Mac`.  Skip to the section for your appropriate operating system.

## Step by Step Lab Instructions

### 1.1	- Basic Developer Portal Customisation

In this section, you will leverage a custom prebuilt theme. A theme file is packaged as a zip file that contains a set of predefined files. The theme zip file is what is loaded into the developer portal. The best method for creating a custom theme is to obtain an existing custom theme, rename it and change the files as explained in the table below to meet your customization needs.

|Filename|Description|
|--------|-----------|  
|`overrides.css`|This style sheet overrides the fonts, colors and other defaults from the inherited base IBM API Connect theme. The name of this css file is specified in the theme.info file.|
|`screenshot.png`|The file contains a screenshot of the home page for the custom theme and is used in the appearance settings so you can easily find the theme you want to enable and set as the default theme. When you are finishing completing the edits to the overrides.css and have set up the welcome banner to your satisfaction, you should take a screen shot of the developer portal home page and capture it with Snagit or some other screen capture tool and place the file in the top level directory of the theme file. The name of this screenshot file is specified in the theme.info file.|
|`templete.php`|The template.php file contains your sub-theme's functions to manipulate Drupal's default markup. It is one of the most useful files when creating or modifying Drupal themes. With this file you can modify any theme hooks variables or add your own variables, using preprocess or process functions. Override any theme function: that is, replace a module's default theme function with one you write. Call hook alter functions, which allow you to alter various parts of Drupal's internals, including the render elements in forms. See <http://api.drupal.org> for more information about these functions.| 
|`thinkibm_connect_theme.info`|The .info file is a static text file for defining and configuring a theme. Each line in the.info file is a key-value pair with the key on the left and the value on the right, with an "equals sign" between them (e.g. name = my_theme). The info file naming conventions needs to have the theme name specified as part of the filename. The project key in the .info file contains the name of the theme.|
|`theme-settings.php`|Drupal themes get a number of configurable settings options for free. For example, most provide toggle switches for the search box, site slogans, user pictures, and so on. Similarly, most provide file uploading widgets to add a custom logo or favicon. These settings are easy: Drupal will add them to the theme's configuration page by default, so it takes no extra work. We want to create our own custom setting, however; one that adds a hidden field that contains the current release information to the Theme configuration form. To do that, we'll need to add a new file to the theme: `theme-settings.php`. The function name specified within this file needs to be prefixed with the theme name: `thinkibm_connect_theme`.|
|`thinkibm_welcome_banner.png`|This file provides the image that shows up in the welcome banner. Although the welcome banner does not need to be part of the theme zip file contents, it is useful to place the welcome banner with other theme files.|
|`favicon.ico`|In web development, you can provide a small logo for your site that appears near the address bar and in the bookmarks folder in a visitor's browser. This logo is called the favicon. Drupal provides a default one, which is the recognizable water drop logo. Using the Drupal logo as the favicon is fine but if you really want to make your site stand out, you should provide your own. Favicon files are in the .ico format and are extremely small in dimensions. The default Drupal favicon is 32 pixels high by 32 pixels wide, many browsers use a 16 x 16 pixel version that can be included in the same file. This is because the favicon is only an icon that shows up in the address bar and favorites (bookmarks) list and typically there is not a lot of room there. Any favicon that you create should be just as small.|                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  
|`logo.png`|The default logo that appears at the top left-hand side of the developer portal page.|

#### 1.1.1 - Install and Configure a Custom Theme

Now that we have explained the contents of a custom theme file, it is time to load the custom theme.

1.  Return to the Bluemix API Manager screen.

    Navigate to the Dashboard section and click on the `Sandbox` catalog tile.
    
    Choose the `Settings` tab, followed by the `Portal` option.
    
    Click on the `Portal URL` link to launch the Developer Portal.
    
    ![](https://ibm-apiconnect.github.io/faststart/images/lab7/launch-portal.png)

1.  Login into the developer portal as an administrator using a username of `admin` and the password you set up in Lab 1.

    ![](https://ibm-apiconnect.github.io/faststart/images/lab7/PortalAdminLogin.png)

1.  From the Administrator menu, select `Appearance` and then `Install new theme`.

    ![](https://ibm-apiconnect.github.io/faststart/images/lab7/InstallNewPortalTheme.png)

1.  In the `Install from a URL` field, enter the following URL then click the `Install` button:

    [https://github.com/ibm-apiconnect/pot/raw/gh-pages/assets/lab7/portal-theme.zip](https://ibm-apiconnect.github.io/faststart/assets/lab7/portal-theme.zip)

    ![](https://ibm-apiconnect.github.io/faststart/images/lab7/install-theme.png)
    
1.  Once the installation of the custom theme completes, click the `Enable newly added themes` link.

    ![](https://ibm-apiconnect.github.io/faststart/images/lab7/enable-theme.png)
   
1.  Scroll down the list of themes to find the `thinkibm_connect 7.50` theme.

    Click the `Enable and set default link` link.

    ![](https://ibm-apiconnect.github.io/faststart/images/lab7/set-default-theme.png)
	
1.  Click the `Home` icon in the top-left corner of the screen to return to the home page.

    ![](https://ibm-apiconnect.github.io/faststart/images/lab7/CloseThemeSettings.png)

#### 1.1.2 - Customize the Welcome Banner

Now we want to change the default Welcome banner to use our custom Welcome banner image. 

1.  Navigate to the blocks content page by selecting `Content > Blocks` from the admin menu.

    ![](https://ibm-apiconnect.github.io/faststart/images/lab7/ThinkIBMBlocks.png)

1.  Underneath operations select `Edit` to the right of the Welcome banner block.

    ![](https://ibm-apiconnect.github.io/faststart/images/lab7/edit-welcome-banner.png)

1.  Before you can set the image, you will need to download it to your local computer's disc drive. You can download the file from this URL:

    [https://thinkibm-services.mybluemix.net/portal/welcome-banner.png](https://thinkibm-services.mybluemix.net/portal/welcome-banner.png)

1.  Scroll down the banner edit page, locate the *Image* section and click the `Chose file` button to launch the file explorer.

    ![](https://ibm-apiconnect.github.io/faststart/images/lab7/choose-banner-file.png)

1.  Find the `welcome-banner.png` file. Then, click the `Open` button.

1.  Click the `Upload` button to upload the new welcome banner image.

1.  Scroll to the bottom of the page and click the `Save` button.

1.  Click on the `x` button to close the content block settings.

    ![](https://ibm-apiconnect.github.io/faststart/images/lab7/close-content-block.png)

#### 1.1.3 - Change the Region Settings for the Content Blocks

The region settings for some of the content blocks were reset when we loaded the new theme, so we will set these back.

1.  Navigate to the region settings for block content page by selecting `Structure > Blocks` from the admin menu.

    ![](https://ibm-apiconnect.github.io/faststart/images/lab7/ThinkIBMBlockRegionSettings.png)

1.  Locate the **Sidebar first** block. You will see entries for `Navigation`, `Support` and `User login`.

1.  For **both** the `Navigation` and `User Login` entries, select `- None -` from the drop down menu.

    ![](https://ibm-apiconnect.github.io/faststart/images/lab7/SetBlockRegionstoNone.png)

1.  The only entry left in the section should be `Support`.
	
    ![](https://ibm-apiconnect.github.io/faststart/images/lab7/sidebar-first-complete.png)

1.  Scroll down to the bottom of the screen anc click the `Save Blocks` button.

    ![](https://ibm-apiconnect.github.io/faststart/images/lab7/SaveBlockRegions.png)

1.  Scroll back to the top and click the `x` button to close the block region settings.

    ![](https://ibm-apiconnect.github.io/faststart/images/lab7/close-region-block.png)

1.  You are finished with customizing the developer portal. There is a lot more that can be customized than what we have time for in this lab. Log out of the developer portal.

    ![](https://ibm-apiconnect.github.io/faststart/images/lab7/PortalAdminLogout.png)

### 1.2	- Advanced Developer Portal Customisation

### Adding the slogan
1.	Click Configuration > System > Site Information 
2.	In the 'Slogan' field enter: 'Welcome to the API Portal'

![](https://ibm-apiconnect.github.io/faststart/images/europe2017/lab1/1.png)


3.	Save the configuration 

![](https://ibm-apiconnect.github.io/faststart/images/europe2017/lab1/2.png)
 
4.	Close the current window

![](https://ibm-apiconnect.github.io/faststart/images/europe2017/lab1/3.png)
 
5.	Click Appearance > Settings 
6.	Navigate to the thinkibm_connect theme settings
 
![](https://ibm-apiconnect.github.io/faststart/images/europe2017/lab1/4.png)
 
7.	Expand the Toggle display section and check the box for 'Site Slogan' 

![](https://ibm-apiconnect.github.io/faststart/images/europe2017/lab1/5.png)
 
8.	Save the configuration 
9.	You will notice that you will not be able to see any changes. This is because the color scheme of the slogan is set to white text on the white background. 




### Colour schemes and custom CSS
Here we are going to add custom CSS to our theme. For this we are going to use the ‘Custom CSS’ extension in the developer portal. Note the ‘Custom CSS’ extension is designed to make small CSS changes in the case of emergencies in production. However, the ‘Custom CSS’ extension is also useful to quickly make CSS changes at development time when you are building the look and feel of a developer portal. It saves the need to re-upload the theme every time a new CSS change is made throughout development. Once you are happy with the CSS and it is stable, it should be moved from the ‘Custom CSS’ extension and placed inside the overrides.css file inside the theme itself. 

1.	To activate the ‘Custom CSS’ extension go to Appearance > Setting > thinkibm_connect > Extensions > Check box ‘Custom CSS’.

 ![](https://ibm-apiconnect.github.io/faststart/images/europe2017/lab1/6.png)
      
2.	Save the changes by clicking ‘Save Configuration’ at the bottom of the page. 
3.	Now if you go to Appearance > Setting > thinkibm_connect then scroll down to the menu at the bottom (extensions), there will be a new menu tab called ‘Custom CSS’. In here, any custom CSS declarations can be set. 
4.	Inside the custom CSS editor area, copy the flowing 2 CSS declarations:
             
             /*  CSS for changing the site slogan: */
                h2#site-slogan {
                  padding-top: 1.9em;
                  font-size: 2em !important;
                  color: #444849;

                }
                /* CSS for changing the logo padding: */
                img.site-logo {
                  height: 95px;
                  padding-top: 14px;
                }

![](https://ibm-apiconnect.github.io/faststart/images/europe2017/lab1/7.png)

5.	Now scroll to the bottom of the page and hit ‘Save Configuration’. 
6.	Scroll to the top and hit the X to close the Appearance menu and view the changes you have made. 

![](https://ibm-apiconnect.github.io/faststart/images/europe2017/lab1/8.png)
      

### Custom footer block
1.	Click 'Structure' > 'Blocks' 
2.	Click configure on the 'Developer portal footer' block

![](https://ibm-apiconnect.github.io/faststart/images/europe2017/lab1/9.png)

3.	Open the wysiwyg (button with html on it) editor and paste in the contents below. 

          <div class="footer_wrapper_div"><div class="upper_grey_block"><div class="table_div_left" style="text-align:    center;">&nbsp;</div></div><div class="lower_grey_block"><div class="centered_link" style="text-align: center;"><a>Terms of use |</a>&nbsp;<a>Terms &amp; Conditions</a>&nbsp; |&nbsp;<a>Privacy Policy</a>&nbsp; |&nbsp;<a>Cookies</a>&nbsp;<a>Accessibility</a>&nbsp; |&nbsp;<a>Sitemap</a></div><div class="list_div_left">&nbsp;</div><div class="list_div_left" style="text-align: center;"><a></a>© IBM 2016. All Rights Reserved</div></div></div>

![](https://ibm-apiconnect.github.io/faststart/images/europe2017/lab1/10.png)



4.	Click Update 
5.	Select 'Full HTML' from the 'Text Format' drop down 

![](https://ibm-apiconnect.github.io/faststart/images/europe2017/lab1/11.png)
  
6.	Save the block from the foot of the page
7.	Close the window and view the changes made to the footer. 




### Adding a Custom block to the home page
1.	Click Structure > Pages
2.	Click 'edit' on the 'page-welcome' page (towards the bottom of the list)
3.	Click 'content'

![](https://ibm-apiconnect.github.io/faststart/images/europe2017/lab1/12.png)
    
4.	Using the cog on 'Middle (conditional)' select 'Add content'
5.	Click Custom Blocks
6.	Click on ‘New Custom Content’ and set:
      - Administrative Title: Homepage block 1
      - Title: leave blank

7.	Click on the html button and add in the following html to the body                      
                                        
        <div class="light_grey_homepage_block"><div class="left_side_50_div"><div class="fixed_width_block_div"><h3 class="block_heading_2 align_text_center" style="text-align: center;"><span style="color: #444849;">Create a seamless experience with direct access to our services and intergrate them with your business</span></h3><h3 class="block_heading_2 align_text_center" style="text-align: center;"><span style="color: #444849;">Look through our API products to find the right one for your business. There's dedicated documentaion on hand for you to get started.&nbsp;</span></h3><h3 class="block_heading_2 align_text_center" style="text-align: center;"><span style="color: #444849;">If you have any questions about how to get started or need help with your existing intergration, take a look in our support section.</span></h3></div></div></div>
                      
![](https://ibm-apiconnect.github.io/faststart/images/europe2017/lab1/13.png)

8.	Click update on the html source editor

![](https://ibm-apiconnect.github.io/faststart/images/europe2017/lab1/14.png)
      
9.	Click finish. 
10.	Drag the Homepage block 1 that you just created to the top of the middle section (above the features_apis_title block). This will position this new section on the page underneath the welcome banner.

![](https://ibm-apiconnect.github.io/faststart/images/europe2017/lab1/15.png)


11.	Click 'Update and save'
12.	Close the welcome screen admin menu to see the results on the home page of the portal


### Remove text on the banner image
1.	Click 'Content' > 'Blocks'
2.	On the 'Welcome banner' click edit
3.	Click on the html button and edit the html source to remove all the html which is there currently.

![](https://ibm-apiconnect.github.io/faststart/images/europe2017/lab1/16.png)
  
4.	Click update in the HTML source editor
5.	Scroll down and save the configuration
6.	Close the content editor to see the results on the home page



## Conclusion

**Congratulations!** You have customised your first Developer Portal!

In this lab you learned:

+ How to create a simple LoopBack application
+ How to create a Representational State Transfer (REST) API definition


Proceed to [Lab 2 - Protecting a Payments API using oAuth 2.0](/faststart/europe/2017/lab2.html)
