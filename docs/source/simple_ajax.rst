.. _simple_ajax

Simple AJAX
===========

This guide will walk you through building a simple Pecan web application that uses AJAX to fetch JSON information from a server.

Project Setup
-------------

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

Adding jQuery Support
---------------------

For this project we will need to add `jQuery <http://jquery.com/>`_ support. To add jQuery go into the templates folder and edit the ``layout.html`` file.

Adding jQuery support is easy, we actually only need one line of code:

::

    <script src="http://ajax.googleapis.com/ajax/libs/jquery/2.1.0/jquery.min.js"></script>


The ``layout.html`` file will look like this:

.. code-block:: html

    <html>
        <head>
            <title>${self.title()}</title>
            ${self.style()}
            ${self.javascript()}
            <script src="http://ajax.googleapis.com/ajax/libs/jquery/2.1.0/jquery.min.js"></script>
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

Adding HTML and JavaScript
--------------------------

Let's edit the ``index.html`` file next. We will add the HTML and JavaScript required to have a simple AJAX interaction between the web page and Pecan. Modify ``index.html`` to look like this:

.. code-block:: html

    <%inherit file="layout.html" />

    <%def name="title()">
    Welcome to Pecan!
    </%def>

    <header>
        <h1><img src="/images/logo.png"/></h1>
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
            function onSuccess(data, status) {
                // Use a template or something here instead
                // Just for demo purposes
                $("#result").html("<div>" +
                        "<p></p><strong>Project Name: " + data.name + "</strong></p>" +
                        "<p>Project License: " + data.licensing + "</p>" +
                        "<p><a href='" + data.repository + "'>Project Repository: " + data.repository + "</a></p>" +
                        "<p><a href='" + data.documentation + "'>Project Documentation: " + data.documentation + "</a></p>" +
                        "</div>");
            }

            function onError(data, status) {
                // Handle an error
            }

            $(document).ready(function () {
                $("#submit").click(function () {
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

#. Added a dropdown control and submit button for the user to interact with. They can pick an open source project and get more details on it
#. Added JavaScript to make an AJAX call to the server via an HTTP ``GET`` passing in the ``id`` of the project to fetch more information on
#. Once the ``onSuccess`` event is triggered by the returning data we take that and display it on the web page below the controls

Building the Model with JSON Support
------------------------------------

The HTML and JavaScript work is now taken care of. At this point we can add a model to our project inside of the ``model`` folder. Create a file in there called ``projects.py`` and add the following to it:

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

#. Created a model called ``Project`` that can hold project specific data
#. Added a ``__json__`` method so an instance of the ``Project class`` can be easily represented as JSON. The controller we will soon build will make use of that JSON capability

**Note:** There are other ways to return JSON with Pecan, check out :ref:`jsonify` for more information.

Working with the Controllers
----------------------------

We don't need to really do anything major to the ``root.py`` file in the ``controllers`` folder except to add support for a new controller we will call ``ProjectsController``. Modify the ``root.py`` like this:

::

    from pecan import expose
    
    from myajax.controllers.projects import ProjectsController
    
    
    class RootController(object):
    
        projects = ProjectsController()
    
        @expose(generic=True, template='index.html')
        def index(self):
            return dict()
            
**What did we just do?**

#. Removed some of the initial boilerplate code since we won't be using it
#. Add support for the upcoming ``ProjectsController``

The final piece is to add a file called ``projects.py`` to the ``controllers`` folder. This new file will host the ``ProjectsController`` which will listen for incoming AJAX ``GET`` calls (in our case) and return the appropriate JSON response.

Add the following code to the ``projects.py`` file:

::

    from pecan import expose, response
    from pecan.rest import RestController
    
    from myajax.model.projects import Project
    
    
    class ProjectsController(RestController):
    
        # Note: You would probably store this information in a database
        # This is just for simplicity and demonstration purposes
        def __init__(self):
            self.projects = [
                Project(name='OpenStack',
                        licensing='Apache 2',
                        repository='http://github.com/openstack',
                        documentation='http://docs.openstack.org'),
                Project(name='Pecan',
                        licensing='BSD',
                        repository='http://github.com/stackforge/pecan',
                        documentation='http://pecan.readthedocs.org'),
                Project(name='stevedore',
                        licensing='Apache 2',
                        repository='http://github.com/dreamhost/pecan',
                        documentation='http://stevedore.readthedocs.org')
            ]
    
    
        @expose('json', content_type='application/json')
        def get(self, id):
            # Note: You would want to verify the id doesn't
            # go out of bounds, etc.
            response.status = 200
            #print(self.projects[int(id)])
            return self.projects[int(id)]
            
**What did we just do?**

#. Created a local class variable called ``projects`` that holds three open source projects and their details. Typically this kind of information would probably reside in a database
#. Added code for the new controller that will listen on the ``projects`` endpoint and serve back JSON based on the ``id`` passed in from the web page

Run the application:

::

$ pecan serve config.py

Open a web browser: `http://127.0.0.1:8080/ <http://127.0.0.1:8080/>`_
