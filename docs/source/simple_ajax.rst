Simple AJAX
===========

This guide will walk you through building a simple Pecan web application that uses AJAX to fetch information from the server.

First off, you'll need to install Pecan:

::

$ pip install pecan

Use Pecan's basic template support to start a new project:

::

$ pecan create myajax
$ cd myajax

Put the new project into development mode:

::

$ python setup.py develop

For this project will need to add `jQuery <http://jquery.com/>`_ support to this project. To add jQuery support go into the templates folder and edit the ``layout.html`` file. Modify it so that it resembles this:

::

<html>
        <head>
            <title>${self.title()}</title>
            ${self.style()}
            ${self.javascript()}
            <script src="//ajax.googleapis.com/ajax/libs/jquery/1.11.0/jquery.min.js"></script>
        </head>
        <body>
            ${self.body()}
        </body>
    </html>
    
    <%def name="title()">
        Default Title
    </%def>
    
    <%def name="style()">
        <link rel="stylesheet" type="text/css" media="screen" href="/css/style.css" />
    </%def>
    
    <%def name="javascript()">
        <script language="text/javascript" src="/javascript/shared.js"></script>
    </%def>
    
**What did we just do?**

#. In the ``head`` section we added jQuery support via the `Google CDN <https://developers.google.com/speed/libraries/devguide>`_.

Let's edit the ``index.html`` file next. We will add the HTML and JavaScript required to have a simple AJAX interaction between the web page and Pecan. Modify ``index.html`` to look like this:

::

<%inherit file="layout.html" />
    
    ## provide definitions for blocks we want to redefine
    <%def name="title()">
        Welcome to Pecan!
    </%def>
    
    ## now define the body of the template
        <header>
            <h1><img src="/images/logo.png" /></h1>
        </header>
    
        <div id="content">
            <p>Select a project to get details:</p>
            <select id="projects">
                <option value="0">OpenStack</option>
                <option value="1">Pecan</option>
                <option value="2">Steve Dore</option>
            </select>
            <button id="submit" type="submit">Submit</button>
    
            <script>
                function onSuccess(data, status)
                {
                    // Messy, use a template or something here instead
                    // Just for demo purposes
                    $("#result").html("<div>" +
                            "<p></p><strong>Project Name: " + data.name + "</strong></p>" +
                            "<p>Project License: " + data.licensing + "</p>" +
                            "<p><a href='" + data.repository + "'>Project Repository: " + data.repository + "</a></p>" +
                            "<p><a href='" + data.documentation + "'>Project Documentation: " + data.documentation + "</a></p>" +
                            "</div>");
                }
    
                function onError(data, status)
                {
                    // Handle an error
                }
    
                $(document).ready(function() {
                    $("#submit").click(function(){
    
                        $.ajax({
                            type: "GET",
                            url: "http://127.0.0.1:8080/projects/",
                            data: "id=" + $("#projects").val(),
                            contentType: 'application/json',
                            dataType: 'json',
                            success: onSuccess,
                            error: onError
                        });
    
                        return false;
                    });
                });
            </script>
    
            <div id="result"></div>
    
        </div>

**What did we just do?**

#. Added a dropdown control and submit button for the user to interact with. They can pick an open source project and getmore details on it
#. Added JavaScript to make an AJAX call to the server via an HTTP GET passing in the **id** of the project we want to fetch more information on
#. Once **onSuccess** is called by the returning data we take that and display it in the **result** div.

At this point the HTML and JavaScript is taken care of. At this point we can add a model to our project inside of the ``model``folder. Create a file there called ``projects.py`` and add the following to it:

::

class Project(object):
        def __init__(self, name, licensing, repository, documentation):
            self.name = name
            self.licensing = licensing
            self.repository = repository
            self.documentation = documentation
    
        def __json__(self):
            return dict(
                name=self.name,
                licensing=self.licensing,
                repository=self.repository,
                documentation=self.documentation
            )
    
**What did we just do?**

#. Created a model called **Project** that can holdproject specific data
#. Added a `__json__ <http://pecan.readthedocs.org/en/latest/jsonify.html>`_ method so an instance of the Project class can be easily represented as JSON. The controller we will soon build will make use of the JSON

