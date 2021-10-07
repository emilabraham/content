---
title: 'Django Tutorial Part 8: User authentication and permissions'
slug: Learn/Server-side/Django/Authentication
tags:
  - Article
  - Authentication
  - Beginner
  - Forms
  - Learn
  - Permissions
  - Python
  - Server
  - Tutorial
  - django
  - django authentication
  - server-side
  - sessions
---
<div>{{LearnSidebar}}</div>

<div>{{PreviousMenuNext("Learn/Server-side/Django/Sessions", "Learn/Server-side/Django/Forms", "Learn/Server-side/Django")}}</div>

<p>In this tutorial, we'll show you how to allow users to log in to your site with their own accounts, and how to control what they can do and see based on whether or not they are logged in and their <em>permissions</em>. As part of this demonstration, we'll extend the <a href="/en-US/docs/Learn/Server-side/Django/Tutorial_local_library_website">LocalLibrary</a> website, adding login and logout pages, and user- and staff-specific pages for viewing books that have been borrowed.</p>

<table>
 <tbody>
  <tr>
   <th scope="row">Prerequisites:</th>
   <td>Complete all previous tutorial topics, up to and including <a href="/en-US/docs/Learn/Server-side/Django/Sessions">Django Tutorial Part 7: Sessions framework</a>.</td>
  </tr>
  <tr>
   <th scope="row">Objective:</th>
   <td>To understand how to set up and use user authentication and permissions.</td>
  </tr>
 </tbody>
</table>

<h2 id="Overview">Overview</h2>

<p>Django provides an authentication and authorization ("permission") system, built on top of the session framework discussed in the <a href="/en-US/docs/Learn/Server-side/Django/Sessions">previous tutorial</a>, that allows you to verify user credentials and define what actions each user is allowed to perform. The framework includes built-in models for <code>Users</code> and <code>Groups</code> (a generic way of applying permissions to more than one user at a time), permissions/flags that designate whether a user may perform a task, forms and views for logging in users, and view tools for restricting content.</p>

<div class="notecard note">
  <p><strong>Note:</strong> According to Django the authentication system aims to be very generic, and so does not provide some features provided in other web authentication systems. Solutions for some common problems are available as third-party packages. For example, throttling of login attempts and authentication against third parties (e.g. OAuth).</p>
</div>

<p>In this tutorial, we'll show you how to enable user authentication in the <a href="/en-US/docs/Learn/Server-side/Django/Tutorial_local_library_website">LocalLibrary</a> website, create your own login and logout pages, add permissions to your models, and control access to pages. We'll use the authentication/permissions to display lists of books that have been borrowed for both users and librarians.</p>

<p>The authentication system is very flexible, and you can build up your URLs, forms, views, and templates from scratch if you like, just calling the provided API to log in the user. However, in this article, we're going to use Django's "stock" authentication views and forms for our login and logout pages. We'll still need to create some templates, but that's pretty easy.</p>

<p>We'll also show you how to create permissions, and check on login status and permissions in both views and templates.</p>

<h2 id="Enabling_authentication">Enabling authentication</h2>

<p>The authentication was enabled automatically when we <a href="/en-US/docs/Learn/Server-side/Django/skeleton_website">created the skeleton website</a> (in tutorial 2) so you don't need to do anything more at this point.</p>

<div class="notecard note">
  <p><strong>Note:</strong> The necessary configuration was all done for us when we created the app using the <code>django-admin startproject</code> command. The database tables for users and model permissions were created when we first called <code>python manage.py migrate</code>.</p>
</div>

<p>The configuration is set up in the <code>INSTALLED_APPS</code> and <code>MIDDLEWARE</code> sections of the project file (<strong>locallibrary/locallibrary/settings.py</strong>), as shown below:</p>

<pre class="brush: python">INSTALLED_APPS = [
    ...
    'django.contrib.auth',  #Core authentication framework and its default models.
    'django.contrib.contenttypes',  #Django content type system (allows permissions to be associated with models).
    ....

MIDDLEWARE = [
    ...
    'django.contrib.sessions.middleware.SessionMiddleware',  #Manages sessions across requests
    ...
    'django.contrib.auth.middleware.AuthenticationMiddleware',  #Associates users with requests using sessions.
    ....
</pre>

<h2 id="Creating_users_and_groups">Creating users and groups</h2>

<p>You already created your first user when we looked at the <a href="/en-US/docs/Learn/Server-side/Django/Admin_site">Django admin site</a> in tutorial 4 (this was a superuser, created with the command <code>python manage.py createsuperuser)</code>. Our superuser is already authenticated and has all permissions, so we'll need to create a test user to represent a normal site user. We'll be using the admin site to create our <em>locallibrary</em> groups and website logins, as it is one of the quickest ways to do so.</p>

<div class="notecard note">
  <p><strong>Note:</strong> You can also create users programmatically, as shown below. You would have to do this, for example, if developing an interface to allow users to create their own logins (you shouldn't give users access to the admin site).</p>

<pre class="brush: python">from django.contrib.auth.models import User

# Create user and save to the database
user = User.objects.create_user('myusername', 'myemail@crazymail.com', 'mypassword')

# Update fields and then save again
user.first_name = 'John'
user.last_name = 'Citizen'
user.save()
</pre>

<p>It is highly recommended to set up a custom user model when starting an actual project. You'll be able to easily customize it in the future if the need arises. For more information, see <a href="https://docs.djangoproject.com/en/3.1/topics/auth/customizing/#using-a-custom-user-model-when-starting-a-project">Using a custom user model when starting</a> (Django docs).</p>

</div>

<p>Below we'll first create a group and then a user. Even though we don't have any permissions to add for our library members yet, if we need to later, it will be much easier to add them once to the group than individually to each member.</p>

<p>Start the development server and navigate to the admin site in your local web browser (<a href="http://127.0.0.1:8000/admin/">http://127.0.0.1:8000/admin/</a>). Login to the site using the credentials for your superuser account. The top level of the Admin site displays all of your models, sorted by "Django application". From the <strong>Authentication and Authorization</strong> section, you can click the <strong>Users</strong> or <strong>Groups</strong> links to see their existing records.</p>

<p><img alt="Admin site - add groups or users" src="admin_authentication_add.png"></p>

<p>First lets create a new group for our library members.</p>

<ol>
 <li>Click the <strong>Add</strong> button (next to Group) to create a new <em>Group</em>; enter the <strong>Name</strong> "Library Members" for the group.
 <img alt="Admin site - add group" src="admin_authentication_add_group.png"></li>
 <li>We don't need any permissions for the group, so just press <strong>SAVE</strong> (you will be taken to a list of groups).</li>
</ol>

<p>Now let's create a user:</p>

<ol>
 <li>Navigate back to the home page of the admin site</li>
 <li>Click the <strong>Add</strong> button next to <em>Users</em> to open the <em>Add user</em> dialog box.<img alt="Admin site - add user pt1" src="admin_authentication_add_user_prt1.png"></li>
 <li>Enter an appropriate <strong>Username</strong> and <strong>Password</strong>/<strong>Password confirmation</strong> for your test user</li>
 <li>Press <strong>SAVE</strong> to create the user.<br>
  <br>
  The admin site will create the new user and immediately take you to a <em>Change user</em> screen where you can change your <strong>username</strong> and add information for the User model's optional fields. These fields include the first name, last name, email address, and the user's status and permissions (only the <strong>Active</strong> flag should be set). Further down you can specify the user's groups and permissions, and see important dates related to the user (e.g. their join date and last login date).
  <img alt="Admin site - add user pt2" src="admin_authentication_add_user_prt2.png"></li>
 <li>In the <em>Groups</em> section, select <strong>Library Member</strong> group from the list of <em>Available groups</em>, and then press the <strong>right-arrow</strong> between the boxes to move it into the <em>Chosen groups</em> box.
   <img alt="Admin site - add user to group" src="admin_authentication_user_add_group.png"></li>
 <li>We don't need to do anything else here, so just select <strong>SAVE</strong> again, to go to the list of users.</li>
</ol>

<p>That's it! Now you have a "normal library member" account that you will be able to use for testing (once we've implemented the pages to enable them to log in).</p>

<div class="notecard note">
  <p><strong>Note:</strong> You should try creating another library member user. Also, create a group for Librarians, and add a user to that too!</p>
</div>

<h2 id="Setting_up_your_authentication_views">Setting up your authentication views</h2>

<p>Django provides almost everything you need to create authentication pages to handle login, log out, and password management "out of the box". This includes a URL mapper, views and forms, but it does not include the templates — we have to create our own!</p>

<p>In this section, we show how to integrate the default system into the <em>LocalLibrary</em> website and create the templates. We'll put them in the main project URLs.</p>

<div class="notecard note">
  <p><strong>Note:</strong> You don't have to use any of this code, but it is likely that you'll want to because it makes things a lot easier. You'll almost certainly need to change the form handling code if you change your user model (an advanced topic!) but even so, you would still be able to use the stock view functions.</p>
</div>

<div class="notecard note">
  <p><strong>Note:</strong> In this case, we could reasonably put the authentication pages, including the URLs and templates, inside our catalog application. However, if we had multiple applications it would be better to separate out this shared login behavior and have it available across the whole site, so that is what we've shown here!</p>
</div>

<h3 id="Project_URLs">Project URLs</h3>
<p>Add the following to the bottom of the project urls.py file (<strong>locallibrary/locallibrary/urls.py</strong>) file:</p>

<pre class="brush: python">#Add Django site authentication urls (for login, logout, password management)

urlpatterns += [
    path('accounts/', include('django.contrib.auth.urls')),
]
</pre>

<p>Navigate to the <a href="http://127.0.0.1:8000/accounts/">http://127.0.0.1:8000/accounts/</a> URL (note the trailing forward slash!) and Django will show an error that it could not find this URL, and listing all the URLs it tried. From this you can see the URLs that will work, for example:</p>

<div class="notecard note">
  <p><strong>Note:</strong> Using the above method adds the following URLs with names in square brackets, which can be used to reverse the URL mappings. You don't have to implement anything else — the above URL mapping automatically maps the below mentioned URLs.</p>

<pre class="brush: python">accounts/ login/ [name='login']
accounts/ logout/ [name='logout']
accounts/ password_change/ [name='password_change']
accounts/ password_change/done/ [name='password_change_done']
accounts/ password_reset/ [name='password_reset']
accounts/ password_reset/done/ [name='password_reset_done']
accounts/ reset/&lt;uidb64&gt;/&lt;token&gt;/ [name='password_reset_confirm']
accounts/ reset/done/ [name='password_reset_complete']</pre>
</div>

<p>Now try to navigate to the login URL (<a href="http://127.0.0.1:8000/accounts/login/">http://127.0.0.1:8000/accounts/login/</a>). This will fail again, but with an error that tells you that we're missing the required template (<strong>registration/login.html</strong>) on the template search path.
You'll see the following lines listed in the yellow section at the top:</p>

<pre class="brush: python">Exception Type:    TemplateDoesNotExist
Exception Value:    registration/login.html</pre>

<p>The next step is to create a registration directory on the search path and then add the <strong>login.html</strong> file.</p>

<h3 id="Template_directory">Template directory</h3>

<p>The URLs (and implicitly, views) that we just added expect to find their associated templates in a directory <strong>/registration/</strong> somewhere in the templates search path.</p>

<p>For this site, we'll put our HTML pages in the <strong>templates/registration/</strong> directory. This directory should be in your project root directory, i.e the same directory as the <strong>catalog</strong> and <strong>locallibrary</strong> folders. Please create these folders now.</p>

<div class="notecard note">
  <p><strong>Note:</strong> Your folder structure should now look like the below:<br>
 locallibrary (Django project folder)<br>
    |_catalog<br>
    |_locallibrary<br>
    |_templates <strong>(new)</strong><br>
                 |_registration</p>
</div>

<p>To make the <strong>templates</strong> directory visible to the template loader we need to add it in the template search path. Open the project settings (<strong>/locallibrary/locallibrary/settings.py</strong>).</p>


<p>Then import the <code>os</code> module (add the following line near the top of the file).</p>
    <pre class="brush: python">import os # needed by code below</pre>

<p>Update the <code>TEMPLATES</code> section's <code>'DIRS'</code> line as shown:</p>

<pre class="brush: python">    ...
    TEMPLATES = [
      {
       ...
       'DIRS': [os.path.join(BASE_DIR, 'templates')],
       'APP_DIRS': True,
       ...
    </pre>



<h3 id="Login_template">Login template</h3>

<div class="notecard warning">
<p><strong>Warning:</strong> The authentication templates provided in this article are a very basic/slightly modified version of the Django demonstration login templates. You may need to customise them for your own use!</p>
</div>

<p>Create a new HTML file called /<strong>locallibrary/templates/registration/login.html</strong> and give it the following contents:</p>

<pre class="brush: html">{% extends "base_generic.html" %}

{% block content %}

  {% if form.errors %}
    &lt;p&gt;Your username and password didn't match. Please try again.&lt;/p&gt;
  {% endif %}

  {% if next %}
    {% if user.is_authenticated %}
      &lt;p&gt;Your account doesn't have access to this page. To proceed,
      please login with an account that has access.&lt;/p&gt;
    {% else %}
      &lt;p&gt;Please login to see this page.&lt;/p&gt;
    {% endif %}
  {% endif %}

  &lt;form method="post" action="{% url 'login' %}"&gt;
    {% csrf_token %}
    &lt;table&gt;
      &lt;tr&gt;
        &lt;td&gt;\{{ form.username.label_tag }}&lt;/td&gt;
        &lt;td&gt;\{{ form.username }}&lt;/td&gt;
      &lt;/tr&gt;
      &lt;tr&gt;
        &lt;td&gt;\{{ form.password.label_tag }}&lt;/td&gt;
        &lt;td&gt;\{{ form.password }}&lt;/td&gt;
      &lt;/tr&gt;
    &lt;/table&gt;
    &lt;input type="submit" value="login" /&gt;
    &lt;input type="hidden" name="next" value="\{{ next }}" /&gt;
  &lt;/form&gt;

  {# Assumes you setup the password_reset view in your URLconf #}
  &lt;p&gt;&lt;a href="{% url 'password_reset' %}"&gt;Lost password?&lt;/a&gt;&lt;/p&gt;

{% endblock %}</pre>

<p>This template shares some similarities with the ones we've seen before — it extends our base template and overrides the <code>content</code> block. The rest of the code is fairly standard form handling code, which we will discuss in a later tutorial. All you need to know for now is that this will display a form in which you can enter your username and password, and that if you enter invalid values you will be prompted to enter correct values when the page refreshes.</p>

<p>Navigate back to the login page (<a href="http://127.0.0.1:8000/accounts/login/">http://127.0.0.1:8000/accounts/login/</a>) once you've saved your template, and you should see something like this:</p>

<p><img alt="Library login page v1" src="library_login.png"></p>

<p>If you log in using valid credentials, you'll be redirected to another page (by default this will be <a href="http://127.0.0.1:8000/accounts/profile/">http://127.0.0.1:8000/accounts/profile/</a>). The problem is that, by default, Django expects that upon logging in you will want to be taken to a profile page, which may or may not be the case. As you haven't defined this page yet, you'll get another error!</p>

<p>Open the project settings (<strong>/locallibrary/locallibrary/settings.py</strong>) and add the text below to the bottom. Now when you log in you should be redirected to the site homepage by default.</p>

<pre class="brush: python"># Redirect to home URL after login (Default redirects to /accounts/profile/)
LOGIN_REDIRECT_URL = '/'
</pre>

<h3 id="Logout_template">Logout template</h3>

<p>If you navigate to the logout URL (<a href="http://127.0.0.1:8000/accounts/logout/">http://127.0.0.1:8000/accounts/logout/</a>) then you'll see some odd behavior — your user will be logged out sure enough, but you'll be taken to the <strong>Admin</strong> logout page. That's not what you want, if only because the login link on that page takes you to the Admin login screen (and that is only available to users who have the <code>is_staff</code> permission).</p>

<p>Create and open /<strong>locallibrary/templates/registration/logged_out.html</strong>. Copy in the text below:</p>

<pre class="brush: html">{% extends "base_generic.html" %}

{% block content %}
  &lt;p&gt;Logged out!&lt;/p&gt;
  &lt;a href="{% url 'login'%}"&gt;Click here to login again.&lt;/a&gt;
{% endblock %}</pre>

<p>This template is very simple. It just displays a message informing you that you have been logged out, and provides a link that you can press to go back to the login screen. If you go to the logout URL again you should see this page:</p>

<p><img alt="Library logout page v1" src="library_logout.png"></p>

<h3 id="Password_reset_templates">Password reset templates</h3>

<p>The default password reset system uses email to send the user a reset link. You need to create forms to get the user's email address, send the email, allow them to enter a new password, and to note when the whole process is complete.</p>

<p>The following templates can be used as a starting point.</p>

<h4 id="Password_reset_form">Password reset form</h4>

<p>This is the form used to get the user's email address (for sending the password reset email). Create <strong>/locallibrary/templates/registration/password_reset_form.html</strong>, and give it the following contents:</p>

<pre class="brush: html">{% extends "base_generic.html" %}

{% block content %}
  &lt;form action="" method="post"&gt;
  {% csrf_token %}
  {% if form.email.errors %}
    \{{ form.email.errors }}
  {% endif %}
      &lt;p&gt;\{{ form.email }}&lt;/p&gt;
    &lt;input type="submit" class="btn btn-default btn-lg" value="Reset password"&gt;
  &lt;/form&gt;
{% endblock %}
</pre>

<h4 id="Password_reset_done">Password reset done</h4>

<p>This form is displayed after your email address has been collected. Create <strong>/locallibrary/templates/registration/password_reset_done.html</strong>, and give it the following contents:</p>

<pre class="brush: html">{% extends "base_generic.html" %}

{% block content %}
  &lt;p&gt;We've emailed you instructions for setting your password. If they haven't arrived in a few minutes, check your spam folder.&lt;/p&gt;
{% endblock %}
</pre>

<h4 id="Password_reset_email">Password reset email</h4>

<p>This template provides the text of the HTML email containing the reset link that we will send to users. Create <strong>/locallibrary/templates/registration/password_reset_email.html</strong>, and give it the following contents:</p>

<pre class="brush: html">Someone asked for password reset for email \{{ email }}. Follow the link below:
\{{ protocol}}://\{{ domain }}{% url 'password_reset_confirm' uidb64=uid token=token %}
</pre>

<h4 id="Password_reset_confirm">Password reset confirm</h4>

<p>This page is where you enter your new password after clicking the link in the password reset email. Create <strong>/locallibrary/templates/registration/password_reset_confirm.html</strong>, and give it the following contents:</p>

<pre class="brush: html">{% extends "base_generic.html" %}

{% block content %}
    {% if validlink %}
        &lt;p&gt;Please enter (and confirm) your new password.&lt;/p&gt;
        &lt;form action="" method="post"&gt;
        {% csrf_token %}
            &lt;table&gt;
                &lt;tr&gt;
                    &lt;td&gt;\{{ form.new_password1.errors }}
                        &lt;label for="id_new_password1"&gt;New password:&lt;/label&gt;&lt;/td&gt;
                    &lt;td&gt;\{{ form.new_password1 }}&lt;/td&gt;
                &lt;/tr&gt;
                &lt;tr&gt;
                    &lt;td&gt;\{{ form.new_password2.errors }}
                        &lt;label for="id_new_password2"&gt;Confirm password:&lt;/label&gt;&lt;/td&gt;
                    &lt;td&gt;\{{ form.new_password2 }}&lt;/td&gt;
                &lt;/tr&gt;
                &lt;tr&gt;
                    &lt;td&gt;&lt;/td&gt;
                    &lt;td&gt;&lt;input type="submit" value="Change my password" /&gt;&lt;/td&gt;
                &lt;/tr&gt;
            &lt;/table&gt;
        &lt;/form&gt;
    {% else %}
        &lt;h1&gt;Password reset failed&lt;/h1&gt;
        &lt;p&gt;The password reset link was invalid, possibly because it has already been used. Please request a new password reset.&lt;/p&gt;
    {% endif %}
{% endblock %}
</pre>

<h4 id="Password_reset_complete">Password reset complete</h4>

<p>This is the last password-reset template, which is displayed to notify you when the password reset has succeeded. Create <strong>/locallibrary/templates/registration/password_reset_complete.html</strong>, and give it the following contents:</p>

<pre class="brush: html">{% extends "base_generic.html" %}

{% block content %}
  &lt;h1&gt;The password has been changed!&lt;/h1&gt;
  &lt;p&gt;&lt;a href="{% url 'login' %}"&gt;log in again?&lt;/a&gt;&lt;/p&gt;
{% endblock %}</pre>

<h3 id="Testing_the_new_authentication_pages">Testing the new authentication pages</h3>

<p>Now that you've added the URL configuration and created all these templates, the authentication pages should now just work!</p>

<p>You can test the new authentication pages by attempting to log in to and then log out of your superuser account using these URLs:</p>

<ul>
 <li><a href="http://127.0.0.1:8000/accounts/login/">http://127.0.0.1:8000/accounts/login/</a></li>
 <li><a href="http://127.0.0.1:8000/accounts/logout/">http://127.0.0.1:8000/accounts/logout/</a></li>
</ul>

<p>You'll be able to test the password reset functionality from the link in the login page. <strong>Be aware that Django will only send reset emails to addresses (users) that are already stored in its database!</strong></p>

<div class="notecard note">
  <p><strong>Note:</strong> The password reset system requires that your website supports email, which is beyond the scope of this article, so this part <strong>won't work yet</strong>. To allow testing, put the following line at the end of your settings.py file. This logs any emails sent to the console (so you can copy the password reset link from the console).</p>

  <pre class="brush: python">EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'</pre>

  <p>For more information, see <a href="https://docs.djangoproject.com/en/3.1/topics/email/">Sending email</a> (Django docs).</p>
</div>

<h2 id="Testing_against_authenticated_users">Testing against authenticated users</h2>

<p>This section looks at what we can do to selectively control content the user sees based on whether they are logged in or not.</p>

<h3 id="Testing_in_templates">Testing in templates</h3>

<p>You can get information about the currently logged in user in templates with the <code>\{{ user }}</code> template variable (this is added to the template context by default when you set up the project as we did in our skeleton).</p>

<p>Typically you will first test against the <code>\{{ user.is_authenticated }}</code> template variable to determine whether the user is eligible to see specific content. To demonstrate this, next we'll update our sidebar to display a "Login" link if the user is logged out, and a "Logout" link if they are logged in.</p>

<p>Open the base template (<strong>/locallibrary/catalog/templates/base_generic.html</strong>) and copy the following text into the <code>sidebar</code> block, immediately before the <code>endblock</code> template tag.</p>

<pre class="brush: html">  &lt;ul class="sidebar-nav"&gt;

    ...

   {% if user.is_authenticated %}
     &lt;li&gt;User: \{{ user.get_username }}&lt;/li&gt;
     &lt;li&gt;&lt;a href="{% url 'logout'%}?next=\{{request.path}}"&gt;Logout&lt;/a&gt;&lt;/li&gt;
   {% else %}
     &lt;li&gt;&lt;a href="{% url 'login'%}?next=\{{request.path}}"&gt;Login&lt;/a&gt;&lt;/li&gt;
   {% endif %}
  &lt;/ul&gt;</pre>

<p>As you can see, we use <code>if</code>-<code>else</code>-<code>endif</code> template tags to conditionally display text based on whether <code>\{{ user.is_authenticated }}</code> is true. If the user is authenticated then we know that we have a valid user, so we call <code>\{{ user.get_username }}</code> to display their name.</p>

<p>We create the login and logout link URLs using the <code>url</code> template tag and the names of the respective URL configurations. Note also how we have appended <code>?next=\{{request.path}}</code> to the end of the URLs. What this does is add a URL parameter <code>next</code> containing the address (URL) of the <em>current</em> page, to the end of the linked URL. After the user has successfully logged in/out, the views will use this "<code>next</code>" value to redirect the user back to the page where they first clicked the login/logout link.</p>

<div class="notecard note">
  <p><strong>Note:</strong> Try it out! If you're on the home page and you click Login/Logout in the sidebar, then after the operation completes you should end up back on the same page.</p>
</div>

<h3 id="Testing_in_views">Testing in views</h3>

<p>If you're using function-based views, the easiest way to restrict access to your functions is to apply the <code>login_required</code> decorator to your view function, as shown below. If the user is logged in then your view code will execute as normal. If the user is not logged in, this will redirect to the login URL defined in the project settings (<code>settings.LOGIN_URL</code>), passing the current absolute path as the <code>next</code> URL parameter. If the user succeeds in logging in then they will be returned back to this page, but this time authenticated.</p>

<pre class="brush: python">from django.contrib.auth.decorators import login_required

@login_required
def my_view(request):
    ...</pre>

<div class="notecard note">
  <p><strong>Note:</strong> You can do the same sort of thing manually by testing on <code>request.user.is_authenticated</code>, but the decorator is much more convenient!</p>
</div>

<p>Similarly, the easiest way to restrict access to logged-in users in your class-based views is to derive from <code>LoginRequiredMixin</code>. You need to declare this mixin first in the superclass list, before the main view class.</p>

<pre class="brush: python">from django.contrib.auth.mixins import LoginRequiredMixin

class MyView(LoginRequiredMixin, View):
    ...</pre>

<p>This has exactly the same redirect behavior as the <code>login_required</code> decorator. You can also specify an alternative location to redirect the user to if they are not authenticated (<code>login_url</code>), and a URL parameter name instead of "<code>next</code>" to insert the current absolute path (<code>redirect_field_name</code>).</p>

<pre class="brush: python">class MyView(LoginRequiredMixin, View):
    login_url = '/login/'
    redirect_field_name = 'redirect_to'
</pre>

<p>For additional detail, check out the <a href="https://docs.djangoproject.com/en/3.1/topics/auth/default/#limiting-access-to-logged-in-users">Django docs here</a>.</p>

<h2 id="Example_—_listing_the_current_users_books">Example — listing the current user's books</h2>

<p>Now that we know how to restrict a page to a particular user, let's create a view of the books that the current user has borrowed.</p>

<p>Unfortunately, we don't yet have any way for users to borrow books! So before we can create the book list we'll first extend the <code>BookInstance</code> model to support the concept of borrowing and use the Django Admin application to loan a number of books to our test user.</p>

<h3 id="Models">Models</h3>

<p>First, we're going to have to make it possible for users to have a <code>BookInstance</code> on loan (we already have a <code>status</code> and a <code>due_back</code> date, but we don't yet have any association between this model and a User. We'll create one using a <code>ForeignKey</code> (one-to-many) field. We also need an easy mechanism to test whether a loaned book is overdue.</p>

<p>Open <strong>catalog/models.py</strong>, and import the <code>User</code> model from <code>django.contrib.auth.models</code> (add this just below the previous import line at the top of the file, so <code>User</code> is available to subsequent code that makes use of it):</p>

<pre class="brush: python">from django.contrib.auth.models import User
</pre>

<p>Next, add the <code>borrower</code> field to the <code>BookInstance</code> model:</p>

<pre class="brush: python">borrower = models.ForeignKey(User, on_delete=models.SET_NULL, null=True, blank=True)
</pre>

<p>While we're here, let's add a property that we can call from our templates to tell if a particular book instance is overdue. While we could calculate this in the template itself, using a <a href="https://docs.python.org/3/library/functions.html#property">property</a> as shown below will be much more efficient.</p>

<p>Add this somewhere near the top of the file:</p>

<pre class="brush: python">from datetime import date</pre>

<p>Now add the following property definition to the <code>BookInstance</code> class:</p>

<pre class="brush: python">@property
def is_overdue(self):
    if self.due_back and date.today() &gt; self.due_back:
        return True
    return False</pre>

<div class="notecard note">
  <p><strong>Note:</strong> We first verify whether <code>due_back</code> is empty before making a comparison. An empty <code>due_back</code> field would cause Django to throw an error instead of showing the page: empty values are not comparable. This is not something we would want our users to experience!</p>
</div>

<p>Now that we've updated our models, we'll need to make fresh migrations on the project and then apply those migrations:</p>

<pre class="brush: bash">python3 manage.py makemigrations
python3 manage.py migrate
</pre>

<h3 id="Admin">Admin</h3>

<p>Now open <strong>catalog/admin.py</strong>, and add the <code>borrower</code> field to the <code>BookInstanceAdmin</code> class in both the <code>list_display</code> and the <code>fieldsets</code> as shown below.
This will make the field visible in the Admin section, allowing us to assign a <code>User</code> to a <code>BookInstance</code> when needed.</p>

<pre class="brush: python">@admin.register(BookInstance)
class BookInstanceAdmin(admin.ModelAdmin):
    list_display = ('book', 'status', 'borrower', 'due_back', 'id')
    list_filter = ('status', 'due_back')

    fieldsets = (
        (None, {
            'fields': ('book','imprint', 'id')
        }),
        ('Availability', {
            'fields': ('status', 'due_back','borrower')
        }),
    )</pre>

<h3 id="Loan_a_few_books">Loan a few books</h3>

<p>Now that it's possible to loan books to a specific user, go and loan out a number of <code>BookInstance</code> records. Set their <code>borrowed</code> field to your test user, make the <code>status</code> "On loan", and set due dates both in the future and the past.</p>

<div class="notecard note">
  <p><strong>Note:</strong> We won't spell the process out, as you already know how to use the Admin site!</p>
</div>

<h3 id="On_loan_view">On loan view</h3>

<p>Now we'll add a view for getting the list of all books that have been loaned to the current user. We'll use the same generic class-based list view we're familiar with, but this time we'll also import and derive from <code>LoginRequiredMixin</code>, so that only a logged in user can call this view. We will also choose to declare a <code>template_name</code>, rather than using the default, because we may end up having a few different lists of BookInstance records, with different views and templates.</p>

<p>Add the following to catalog/views.py:</p>

<pre class="brush: python">from django.contrib.auth.mixins import LoginRequiredMixin

class LoanedBooksByUserListView(LoginRequiredMixin,generic.ListView):
    """Generic class-based view listing books on loan to current user."""
    model = BookInstance
    template_name ='catalog/bookinstance_list_borrowed_user.html'
    paginate_by = 10

    def get_queryset(self):
        return BookInstance.objects.filter(borrower=self.request.user).filter(status__exact='o').order_by('due_back')</pre>

<p>In order to restrict our query to just the <code>BookInstance</code> objects for the current user, we re-implement <code>get_queryset()</code> as shown above. Note that "o" is the stored code for "on loan" and we order by the <code>due_back</code> date so that the oldest items are displayed first.</p>

<h3 id="URL_conf_for_on_loan_books">URL conf for on loan books</h3>

<p>Now open <strong>/catalog/urls.py</strong> and add a <code>path()</code> pointing to the above view (you can just copy the text below to the end of the file).</p>

<pre class="brush: python">urlpatterns += [
    path('mybooks/', views.LoanedBooksByUserListView.as_view(), name='my-borrowed'),
]</pre>

<h3 id="Template_for_on-loan_books">Template for on-loan books</h3>

<p>Now, all we need to do for this page is add a template. First, create the template file <strong>/catalog/templates/catalog/bookinstance_list_borrowed_user.html</strong> and give it the following contents:</p>

<pre class="brush: python">{% extends "base_generic.html" %}

{% block content %}
    &lt;h1&gt;Borrowed books&lt;/h1&gt;

    {% if bookinstance_list %}
    &lt;ul&gt;

      {% for bookinst in bookinstance_list %}
      &lt;li class="{% if bookinst.is_overdue %}text-danger{% endif %}"&gt;
        &lt;a href="{% url 'book-detail' bookinst.book.pk %}"&gt;\{{bookinst.book.title}}&lt;/a&gt; (\{{ bookinst.due_back }})
      &lt;/li&gt;
      {% endfor %}
    &lt;/ul&gt;

    {% else %}
      &lt;p&gt;There are no books borrowed.&lt;/p&gt;
    {% endif %}
{% endblock %}</pre>

<p>This template is very similar to those we've created previously for the <code>Book</code> and <code>Author</code> objects. The only thing "new" here is that we check the method we added in the model <code>(bookinst.is_overdue</code>) and use it to change the color of overdue items.</p>

<p>When the development server is running, you should now be able to view the list for a logged in user in your browser at <a href="http://127.0.0.1:8000/catalog/mybooks/">http://127.0.0.1:8000/catalog/mybooks/</a>. Try this out with your user logged in and logged out (in the second case, you should be redirected to the login page).</p>

<h3 id="Add_the_list_to_the_sidebar">Add the list to the sidebar</h3>

<p>The very last step is to add a link for this new page into the sidebar. We'll put this in the same section where we display other information for the logged in user.</p>

<p>Open the base template (<strong>/locallibrary/catalog/templates/base_generic.html</strong>) and add the "My Borrowed" line to the sidebar in the position shown below.</p>

<pre class="brush: python"> &lt;ul class="sidebar-nav"&gt;
   {% if user.is_authenticated %}
   &lt;li&gt;User: \{{ user.get_username }}&lt;/li&gt;

   &lt;li&gt;&lt;a href="{% url 'my-borrowed' %}"&gt;My Borrowed&lt;/a&gt;&lt;/li&gt;

   &lt;li&gt;&lt;a href="{% url 'logout'%}?next=\{{request.path}}"&gt;Logout&lt;/a&gt;&lt;/li&gt;
   {% else %}
   &lt;li&gt;&lt;a href="{% url 'login'%}?next=\{{request.path}}"&gt;Login&lt;/a&gt;&lt;/li&gt;
   {% endif %}
 &lt;/ul&gt;
</pre>

<h3 id="What_does_it_look_like">What does it look like?</h3>

<p>When any user is logged in, they'll see the <em>My Borrowed</em> link in the sidebar, and the list of books displayed as below (the first book has no due date, which is a bug we hope to fix in a later tutorial!).</p>

<p><img alt="Library - borrowed books by user" src="library_borrowed_by_user.png"></p>

<h2 id="Permissions">Permissions</h2>

<p>Permissions are associated with models and define the operations that can be performed on a model instance by a user who has the permission. By default, Django automatically gives <em>add</em>, <em>change</em>, and <em>delete</em> permissions to all models, which allow users with the permissions to perform the associated actions via the admin site. You can define your own permissions to models and grant them to specific users. You can also change the permissions associated with different instances of the same model.</p>

<p>Testing on permissions in views and templates is then very similar for testing on the authentication status (and in fact, testing for a permission also tests for authentication).</p>

<h3 id="Models_2">Models</h3>

<p>Defining permissions is done on the model "<code>class Meta</code>" section, using the <code>permissions</code> field.
You can specify as many permissions as you need in a tuple, each permission itself being defined in a nested tuple containing the permission name and permission display value.
For example, we might define a permission to allow a user to mark that a book has been returned as shown:</p>

<pre class="brush: python">class BookInstance(models.Model):
    ...
    class Meta:
        ...
        permissions = (("can_mark_returned", "Set book as returned"),)</pre>

<p>We could then assign the permission to a "Librarian" group in the Admin site.</p>

<p>Open the <strong>catalog/models.py</strong>, and add the permission as shown above. You will need to re-run your migrations (call <code>python3 manage.py makemigrations</code> and <code>python3 manage.py migrate</code>) to update the database appropriately.</p>

<h3 id="Templates">Templates</h3>

<p>The current user's permissions are stored in a template variable called <code>\{{ perms }}</code>. You can check whether the current user has a particular permission using the specific variable name within the associated Django "app" — e.g. <code>\{{ perms.catalog.can_mark_returned }}</code> will be <code>True</code> if the user has this permission, and <code>False</code> otherwise. We typically test for the permission using the template <code>{% if %}</code> tag as shown:</p>

<pre class="brush: python">{% if perms.catalog.can_mark_returned %}
    &lt;!-- We can mark a BookInstance as returned. --&gt;
    &lt;!-- Perhaps add code to link to a "book return" view here. --&gt;
{% endif %}
</pre>

<h3 id="Views">Views</h3>

<p>Permissions can be tested in function view using the <code>permission_required</code> decorator or in a class-based view using the <code>PermissionRequiredMixin</code>. The pattern are the same as for login authentication, though of course, you might reasonably have to add multiple permissions.</p>

<p>Function view decorator:</p>

<pre class="brush: python">from django.contrib.auth.decorators import permission_required

@permission_required('catalog.can_mark_returned')
@permission_required('catalog.can_edit')
def my_view(request):
    ...</pre>

<p>A permission-required mixin for class-based views.</p>

<pre class="brush: python">from django.contrib.auth.mixins import PermissionRequiredMixin

class MyView(PermissionRequiredMixin, View):
    permission_required = 'catalog.can_mark_returned'
    # Or multiple permissions
    permission_required = ('catalog.can_mark_returned', 'catalog.can_edit')
    # Note that 'catalog.can_edit' is just an example
    # the catalog application doesn't have such permission!</pre>

<div class="notecard note">
  <p><strong>Note:</strong> There is a small default difference in the behavior above. By <strong>default</strong> for a logged-in user with a permission violation:</p>

<ul>
 <li><code>@permission_required</code> redirects to login screen (HTTP Status 302).</li>
 <li><code>PermissionRequiredMixin</code> returns 403 (HTTP Status Forbidden).</li>
</ul>

<p>Normally you will want the <code>PermissionRequiredMixin</code> behavior: return 403 if a user is logged in but does not have the correct permission. To do this for a function view use <code>@login_required</code> and <code>@permission_required</code> with <code>raise_exception=True</code> as shown:</p>

<pre class="brush: python">from django.contrib.auth.decorators import login_required, permission_required

@login_required
@permission_required('catalog.can_mark_returned', raise_exception=True)
def my_view(request):
    ...</pre>
</div>

<h3 id="Example">Example</h3>

<p>We won't update the <em>LocalLibrary</em> here; perhaps in the next tutorial!</p>

<h2 id="Challenge_yourself">Challenge yourself</h2>

<p>Earlier in this article, we showed you how to create a page for the current user listing the books that they have borrowed. The challenge now is to create a similar page that is only visible for librarians, that displays <em>all</em> books that have been borrowed, and which includes the name of each borrower.</p>

<p>You should be able to follow the same pattern as for the other view. The main difference is that you'll need to restrict the view to only librarians. You could do this based on whether the user is a staff member (function decorator: <code>staff_member_required</code>, template variable: <code>user.is_staff</code>) but we recommend that you instead use the <code>can_mark_returned</code> permission and <code>PermissionRequiredMixin</code>, as described in the previous section.</p>

<div class="notecard warning">
  <p><strong>Warning:</strong> Remember not to use your superuser for permissions based testing (permission checks always return true for superusers, even if a permission has not yet been defined!). Instead, create a librarian user, and add the required capability.</p>
</div>

<p>When you are finished, your page should look something like the screenshot below.</p>

<p><img alt="All borrowed books, restricted to librarian" src="library_borrowed_all.png"></p>

<h2 id="Summary">Summary</h2>

<p>Excellent work — you've now created a website that library members can log in into and view their own content and that librarians (with the correct permission) can use to view all loaned books and their borrowers. At the moment we're still just viewing content, but the same principles and techniques are used when you want to start modifying and adding data.</p>

<p>In our next article, we'll look at how you can use Django forms to collect user input, and then start modifying some of our stored data.</p>

<h2 id="See_also">See also</h2>

<ul>
 <li><a href="https://docs.djangoproject.com/en/3.1/topics/auth/">User authentication in Django</a> (Django docs)</li>
 <li><a href="https://docs.djangoproject.com/en/3.1/topics/auth/default/">Using the (default) Django authentication system</a> (Django docs)</li>
 <li><a href="https://docs.djangoproject.com/en/3.1/topics/class-based-views/intro/#decorating-class-based-views">Introduction to class-based views &gt; Decorating class-based views</a> (Django docs)</li>
</ul>

<p>{{PreviousMenuNext("Learn/Server-side/Django/Sessions", "Learn/Server-side/Django/Forms", "Learn/Server-side/Django")}}</p>

<h2 id="In_this_module">In this module</h2>

<ul>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Introduction">Django introduction</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/development_environment">Setting up a Django development environment</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Tutorial_local_library_website">Django Tutorial: The Local Library website</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/skeleton_website">Django Tutorial Part 2: Creating a skeleton website</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Models">Django Tutorial Part 3: Using models</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Admin_site">Django Tutorial Part 4: Django admin site</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Home_page">Django Tutorial Part 5: Creating our home page</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Generic_views">Django Tutorial Part 6: Generic list and detail views</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Sessions">Django Tutorial Part 7: Sessions framework</a></li>
 <li><strong>Django Tutorial Part 8: User authentication and permissions</strong></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Forms">Django Tutorial Part 9: Working with forms</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Testing">Django Tutorial Part 10: Testing a Django web application</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/Deployment">Django Tutorial Part 11: Deploying Django to production</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/web_application_security">Django web application security</a></li>
 <li><a href="/en-US/docs/Learn/Server-side/Django/django_assessment_blog">DIY Django mini blog</a></li>
</ul>