Postmortem
 
 
Incident Analysis
 
After the release of ALX School's System Engineering & Dev Ops project 0x19, an outage occurred on a separate Ubuntu 14.04 container that was running an Apache web server. The server experienced 500 Internal Server Errors instead of providing the expected response of an HTML file defining a basic Holberton WordPress site.
 
Debugging Process
1. Checked the running processes using the command ps aux and verified that two apache2 processes (root and www-data) were running correctly.
2. Examined the sites-available folder in the /etc/apache2/ directory and determined that the web server was serving content from /var/www/html/.
3. Ran strace on the PID of the root Apache process in one terminal while using curl to test the server in another terminal. Unfortunately, the strace output did not provide useful information.
4. Repeated step 3, but this time on the PID of the www-data process. Set lower expectations, but fortunately, strace revealed an error (-1 ENOENT) indicating that the file /var/www/html/wp-includes/class-wp-locale.phpp was not found.
5. Reviewed the files in the /var/www/html/ directory one by one, using Vim pattern matching to locate the incorrect .phpp file extension. Discovered the error in the wp-settings.php file (Line 137: require_once( ABSPATH . WPINC . '/class-wp-locale.php' );).
6. Corrected the issue by removing the trailing p from the line.
7. Tested the server with another curl command, and this time received a successful response (status code 200).
8. Developed a Puppet manifest to automate the resolution of this error.
 
Summary
In summary, the outage was caused by a simple typographical error. The WordPress application encountered a critical error in wp-settings.php while attempting to load the file class-wp-locale.phpp, whereas the correct file name should have been class-wp-locale.php located in the wp-content directory of the application folder. The fix involved removing the extra p from the filename.
 
Prevention
To avoid similar outages in the future, the following preventive measures are recommended:
Conduct thorough testing of the application prior to deployment to identify and address errors beforehand.
In response to this incident, a Puppet manifest named 0x17-web_stack_debugging_3 was created to automate the resolution of identical errors. The manifest replaces any phpp extensions in the file /var/www/html/wp-settings.php with php.

