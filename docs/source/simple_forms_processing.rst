Simple Forms Processing with Pecan
==================================

This guide will walk you through building a simple Pecan web application that will do some simple forms processing.

First off, you'll need to install Pecan:

::

$ pip install pecan

Use Pecan's basic template support to start a new project:

::

$ pecan create mywebsite
$ cd mywebsite

Put the new project into development mode:

::

$ python setup.py develop

With the project ready, go into the templates folder and edit the ``index.html`` file. Modify it so that it resembles this:

::

    <%inherit file="layout.html" />

    <%def name="title()">
        Welcome to Pecan!
    </%def>
        <header>
            <h1><img src="/images/logo.png" /></h1>
        </header>
        <div id="content">
            <form method="POST" action="/">
                <fieldset>
                    <p>Enter a message: <input name="message" /></p>
                    <p>Enter your first name: <input name="first_name" /></p>
                    <input type="submit" value="Submit" />
                </fieldset>
            </form>
            % if not form_post_data is UNDEFINED:
                <p>${form_post_data['first_name']}, your message is: ${form_post_data['message']}</p>
            % endif
        </div>

**What did we just do?**

#. Modified the contents of the ``form`` tag to have two ``input`` tags. The first is named ``message`` and the second is named ``first_name``
#. Added a check if ``form_post_data`` has not been defined so we don't show the message or wording
#. Added code to display the message from the user's ``POST`` action
 
Go into the ``controllers`` folder now and edit the ``root.py`` file. There will be two functions inside of the ``RootController`` class which will display the ``index.html`` file when your web browser hits the ``'/
