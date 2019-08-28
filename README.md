# Deploying a Node Web Server to Heroku

This guide walks through the necessary steps to deploy your full stack Node.js application to Heroku!

### Prerequisites

1. To begin with, you'll need a git repository initialized locally with your basic web server code working and committed. There are a couple of ways to do this.

   * **A: Already using GitHub with this project?**: Sweet!  If you cloned from a remote repository and then wrote/committed your code to the local clone, you should be good to go.

   * **B: I am not using a GitHub repo:** If you haven't set up a git repository for your files yet (or didn't clone), run `git init` locally in the folder with your web server files. 

2. **IF YOUR PROJECT WILL BE USING A DATABASE:** Look at the place in your project where you connect to the database and make sure it has configurations for production. Heroku helpfully provides us with some environment variables we can use for this purpose.
  * **Mongo** will have process.env.MONGODB_URI 
  * **mySQL** will have process.env.JAWSDB_URL
  * For extra help, see the supplemental section [Database Connection Code Examples](#database-connection-code-examples) 

2. **MAKE SURE YOUR SERVER RUNS LOCALLY!** If the server doesn't start up on your machine without crashing, it will not run on Heroku either.

3. Be sure to commit all changes (if you haven't already with the above steps) using `git add .` and `git commit -am "<message>"`. If you haven't run into any errors at this point, you should be able to proceed to the next section.

### Steps to Deploy

1. Log in to Heroku.
   * If you are a windows user open the cmd.exe (NOT Git Bash) and type `heroku login`. Keep this command prompt open in the background. Then, open Git Bash and navigate to the folder with your code.

   * If you are a mac open terminal and type the command `heroku login`. Enter your Heroku credentials and proceed with all the below steps in terminal. Navigate to the folder with your code.

2. Run the command: `git remote –v` .
   * This is to show you that right now, you do not have heroku listed as a remote repository.

3. Run the command `heroku create`.
   * This will create an app instance on the Heroku server and will add heroku as a remote for your local git repository. 

4. Run `git remote –v` again.
   * This isn't necessary, but helps to confirm that Heroku is now in your list of remotes. This time you should see the `heroku` remote.

5. **IF YOUR PROJECT USES A DATABASE (mySQL, Mongo)** -- this is the time to set it up! 
  * **mySQL:** Go to [the Heroku website](https://www.heroku.com/) and click on the name of the project you just created. Under the 'Add-Ons' section, you will add 'JawsDB mySQL'
  * **Mongo:** From the command line in your project directory, run `heroku addons:create mongolab` 

6. **(mySQL WITHOUT SEQUELIZE only)** if you are *NOT* using Sequelize, you will need to do a one-time setup for your database schema / seeds. On a secure connection (not public wifi), use mySQL workbench to connect to your project's JawsDB. Run schema to create the database tables. 
  * *CRITICAL NOTE:* Heroku will NOT allow you do drop or rename the database itself -- you must 'use' the one they provided for you, and you must remove any 'DROP DATABASE IF EXISTS' lines from your schema.

7. Ensure that your `package.json` file is set up correctly. It must have a `start` script and all the project's dependencies defined. E.g.:
   ```json
   {
     "name": "starwars",
     "version": "1.0.0",
     "description": "Helps you find the characters you are looking for",
     "main": "server.js",
     "dependencies": {
        "express": "^4.16.3"
     },
     "scripts": {
       "start": "node server.js"
     }
   }
   ```

8. Ensure your web server is starting with a dynamic port.
   * For an express app, the code for this would look like:

   ```js
   var PORT = process.env.PORT || 3000;
   ...
   app.listen(PORT, function() {
   ```

   * This allows you to get the port from the bound environment variable (using `process.env.PORT`) if it exists, so that when your app starts on heroku's machine it will start listening on the appropriate port.

   * You app will still run on port 3000 locally if you haven't set an environment variable.

9. Ensure that you have at least one HTML page being served at the "/" route. Example:

```js
app.get("/", function(req, res) {
  res.json(path.join(__dirname, "public/index.html"));
});
```

10. Make sure that the application actually works locally. If it doesn't work locally, it won't deploy.

11. Commit any changes you've made up until this point using `git commit -am "<message>"`

12. Run the command `git push heroku master`. A series of processes will be initiated. Once the process is complete note the name of the app.

13. Log in to your Heroku account at www.heroku.com . You will see a list or a (single) app. Note the one that has the same funky name as you saw in bash. Click on it.

14. Click on settings. Then, scroll down until you see the part that says: "Domains". Note the URL listed under Heroku Domain.

15. Finally, go in your browser to the URL listed under the Heroku Domain. If all went well you should see your website!


### Troubleshooting

* If your Heroku app fails to deploy, run the following in your command-line:

  ```
  heroku logs
  ```

  * This should print all the logs produced by both Heroku and your application before the deployment failed. Look for the first indication of an error in the logs. If the error message isn't clear, Google it to learn more or ask an instructor or TA for help.

### Database Connection Code Examples

1. Mongo 
  ```js
  //If the environment variable MONGODB_URI exists, grab that value!
  //Otherwise, use this default we are providing 
  var MONGODB_URI = process.env.MONGODB_URI || "mongodb://localhost/yourlocaldbnamegoeshere";

  mongoose.connect(MONGODB_URI);
  ```
 
2. mySQL (NO SEQUELIZE) 
  ```js
  //if we are on Heroku and the JawsDB addon is configured, we will have the environment variable JAWSDB_URL
  //this variable will contain what mysql package needs to connect :)
  require('dotenv').config();
  var mysql = require('mysql');
  var connection;
  if(process.env.JAWSDB_URL) {  
    connection = mysql.createConnection(process.env.JAWSDB_URL);
  } else {
    //otherwise, we're going to use our local connection!  put your local db set stuff here
    //(and remember our best practice of using the dotenv package and a .env file ;)
    connection = mysql.createConnection({
      host: 'localhost',
      user: process.env.LOCAL_DB_USER,
      password: process.env.LOCAL_DB_PASSWORD,
      database: process.env.LOCAL_DB_DATABASE
    });
  }
  ```

3. mySQL (WITH SEQUELIZE)
  ```js
    //Inside the config.json, be sure that 'production' contains process.env.JAWSDB_URL
    {
    "development": {
      "username": "root", 
      "password": null,
      "database": "exampledb",
      "host": "localhost",
      "dialect": "mysql"
    },
    "test": {
      "username": "root",
      " password": null,
      "database": "testdb",
      "host": "localhost",
      "dialect": "mysql",
      "logging": false
    },
      "production": {
      "use_env_variable": "JAWSDB_URL",
      "dialect": "mysql"
    }
  }
  ```
