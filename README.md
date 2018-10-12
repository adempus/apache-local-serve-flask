# apache-local-serve-flask

So I found a way to host a Flask application over LAN. Useful for testing on mobile devices or whatever use-case demands remote access to your Flask application for testing purposes. You basically need Apache2, WSGI, and gUnicorn to do this. 

## 1. First install Apache2 and WSGI by running the commands: 
	
	```sudo apt-get install apache2
	sudo apt-get install libapache2-mod-wsgi```

Apache is the web server that listens for remote web requests to your Flask app. WSGI is basically a middleware that allows responses to be served from Flask on your local machine through Apache, remotely to another machine on the same network (LAN).

## 2. To check if it's installed correctly, run the command: 

	```hostname -I```

That outputs your IP address. Copy it, then enter it in a browser, and it should present you with an Apache page confirming that it worked!
You can put that same IP address in your mobile device's browser, and it'll also work!! Now Apache is running blind. It needs WSGI to guide it to where your Flask application is, so it can actually serve your app. You need to create a '.wsgi' file in your Flask project's directory for this.

## 3. In the terminal, navigate into your Flask project directory where 'app.py' is located.

## 4. In that location, create a file with a name you'll remember (preferably your project's name), with the following command:
	
	  ```nano projectName.wsgi```

## 5. That command creates a .wsgi file which you can edit in the terminal. Enter the following in the editor: 
	
	```import sys
	sys.path.insert(0, '~/absolutePath/toYour/FlaskProjectFolder')
	from app import app as application```

## 6. Hit 'CTRL X', then 'Y' to save it. 

-In the code above, the second line uses the PATH system to tell Apache where to look for your app via WSGI. (Replace the path in single-quotes with the absolute path to your project's folder) 

-The third line imports the app variable declared in app.py, where: app = Flask(__name__). This way Apache knows just where to look to host your app over LAN.

-To wrap things up, you need to create an Apache config file that points to the '.wsgi' file previously created.

## 7. In the terminal, navigate to Apache's sites-available directory:

	```cd /etc/apache2/sites-available```

This is the folder where config files are stored to tell apache what page to load in the browser when apache starts up. You need to configure it to serve your Flask app. 

## 8. In the terminal, enter command:

	```sudo chown -R yourUsername /etc/apache2```

## 9. In the terminal enter the following command in the /etc/apache2/sites-available folder:

	```nano projectName.conf```

- This creates the config file mentioned earlier to tell apache what site to load. Type the following in the editor:

	  ```<VirtualHost *>
        	ServerName mindbeam.com

        	WSGIScriptAlias / /absolutePath/toYour/FlaskProjectFolder/projectName.wsgi
        	WSGIDaemonProcess app
        	<Directroy /home/adempus/PycharmProjects/project_mindbeam>
                	WSGIProcessGroup projectName
                	WSGIApplicationGroup %{GLOBAL}
                	Order deny,allow
                	Allow from all
        	</Directory>
	  </VirtualHost>```

## 10. Repeat step 6

This basically tells apache that we want to serve responses from our Flask app when we run it with gUnicorn.
To finalize enter in the terminal: 

	```sudo a2dissite 000-default.conf
	sudo a2ensite projectName.conf
	sudo service apache2 reload```

The following commands:
	-removes the default config from Apache's site serving mechanism.
	-adds the config you just created for your Flask app, to Apache's site serving mechanism.
	-reloads Apache to commit the changes.

And well-a you're done. Now to actually serve your Flask app over LAN, navigate to the project's folder, and type the command: 

	```sudo gunicorn -w 2 -b 0.0.0.0:5000 app:projectName```

This will start the Flask server, run it through WSGI, to be served with Apache over LAN. The 0.0.0.0 part is just telling gUnicorn to use your machine's IP address to serve content over port 5000. You can now enter:

	```hostname -I``` 

in your terminal to get your machine's ip address. Type the ip from that command followed by the port like x.x.x.x:5000 in the browser of a separate machine (or mobile device) on the same network as the host machine, and your flask app's default route will be displayed. 
 
