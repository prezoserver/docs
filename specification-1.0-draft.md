# Presentation Server (PrezoServer) Specification Version 1.0

#### Copyright 2014 Kevin Johnston (kevin@prezoserver.org)

## 1.0 Definitions

#### 1.1 Definition of Application

Within the context of the presentation tier, "presentation application" or simply "application" refers to the content and resources that are necessary for building the user interface.

#### 1.2 Definition of Server

A server is defined to be an instance of a web or application server containing a document root directory, designated for a single presentation application, that meets all of the requirements of this specification.

#### 1.3 Definition of File

Files are the resources which produce the user interface. Files may be implemented in one of the following ways:

* Literal - files which exist on the server's file system and produce some aspect of the user interface.

* Virtual - files which do not exist on the server's file system but produce some aspect of the user interface through a server side mechanism. For example, a Java Servlet or an Apache URL Rewrite that produces a JavaScript file for a given URL.

* Wrappers - files which exist on the file system that include another file or server side resource via Server Side Includes (SSI) to produce some aspect of the user interface.  For example, a static JavaScript file which uses SSI to include a JSP, ASP, PHP, or any other dynamic resource that produces the content.

## 2.0 Requirements

#### 2.1 Document Root

The application directory structure is to be placed in the server document root directory. The document root directory should serve content for one and only one application. Generally this requirement can be met using virtual hosts (e.g. Apache HTTP Server), sites (e.g. IIS), or web contexts (e.g. Java).

#### 2.2 Server Side Includes (SSI)

The server must enable SSI for all files with a .html extension. As needed, SSI rules may be enabled for specific files with other extensions and should be limited to file wrappers only.

For portability between PrezoServer implementations SSI must be configured relative the server's document root.

The only SSI command that is to be used within a page is the include command. The virtual path is with respect to the server document root and has the following form:

<!--#include virtual="/<directory>/<file>" -->

#### 2.3 Directory Structure
The application root document directory will contain a folder named "prezo". This directory should only contain the implementation specific files required to meet this specification.  Actual application content should not be placed in this directory.  When packaging an application to deploy on another PrezoServer implementation the prezo directory should not be included. 

The prezo folder will have the following subdirectories and files:

/<document root>
    /prezo
        /fragments
            base.html
        /js
            prezo.js
            csrf.js
    /support
        /fragments
        /js

The following files may be implemented as literal, virtual, or wrapper files:
    /<document root>/prezo/fragments/base.html
    /<document root>/prezo/js/prezo.js
    /<document root>/prezo/js/csrf.js

Regardless of how the files are implemented the following URLs should result in an HTTP response with status code 200 (OK) when accessed by a browser client:

http(s)://<server>:<port>/<document root>/prezo/fragments/base.html
http(s)://<server>:<port>/<document root>/prezo/js/prezo.js
http(s)://<server>:<port>/<document root>/prezo/js/csrf.js

The directory /<document root>/prezo/support/ and any subdirectories and files should not be accessible from a browser client. Any attempt to access the directory or anything under it should result in an HTTP response with status code of 404 (Not Found).

#### 2.4 base.html

The base.html file exists as either a static, virtual, or wrapper file at /<document root>/prezo/fragments/base.html and is accessible by a browser client accessing the URL http(s)://<server>:<port>/<document root>/prezo/fragments/base.html. 

The base.html file contains a base HTML tag which specifies the base href URL. The URL is to be set to http(s)://<server>:<port>/<document root>. If using a reverse proxy server in combination with the presentation server the URL should be set to location of the reverse proxy server, proxy server port, and proxy server path.

#### 2.5 prezo.js

The prezo.js file exists as either a static, virtual, or wrapper file at /<document root>/prezo/js/prezo.js and is accessible by a browser client accessing the URL http(s)://<server>:<port>/<document root>/prezo/js/prezo.js.

This file defines a JavaScript name space and constant for the document root directory of the application. The name space variable and constant is "prezo.application.DOCROOT". If using the presentation server directly (i.e. no reverse proxy server) the variable should be set to "/". For servers that support the notion of web contexts the variable should be set to "<contextPath>/". If a reverse proxy server is being used then the variable should be set to the virtual directory by which the reverse proxy server proxies requests to the presentation server. 

Example 1 (directly accessing presentation server):  
Assuming the URL resembles something like http(s)://server.presentation.com[:<port>]**/**   
prezo.application.DOCROOT = "**/**";

Example 2 (directly accessing presentation server via a context path):  
Assuming the URL resembles something like http(s)://server.presentation.com[:<port>]/**contextPath/**          
prezo.application.DOCROOT = "**contextPath/**";  

Example 3 (using a proxy server):  
Assuming proxy resembles something like:  
http(s)://server.proxy.com[:<port>]/**application/** -> http(s)://server.presentation.com[:<port>]/  
prezo.application.DOCROOT = "**application/**";

#### 2.6 csrf.js

The csrf.js file exists as either a static, virtual, or wrapper file at /<document root>/prezo/js/csrf.js and is accessible by a browser client  accessing the URL http(s)://<server>:<port>/<document root>/prezo/js/csrf.js.

The csrf.js file, when included by the browser client, is to override the XMLHttpRequest object and inject the appropriate server side CSRF variables into the XMLHttpRequest object. The client makes all requests to get or modify data through AJAX. When the request is processed by the presentation server (which is implemented with an application server technology) the server is able to determine if the request has the appropriate CSRF variables and either allow or reject the AJAX request for the data. If the server detects a potential CSRF issue it should return a response body containing "Potential CSRF Attack Detected" with a HTTP status code of 417 to represent this condition. The client, upon detecting the status code, may display appropriate guidance to the user.

The presentation server must also provide a mechanism for including or excluding URL patterns from CSRF protection.
