## Task 1:
You are given an XML file (provided). This file contains application configuration
settings. The application is failing to start because of an error in the XML. Identify and fix the
error. The error could be anything from a syntax issue to a logical problem in the configuration.
Explain the error and your solution.

Upon reviewing the provided XML file, I identified the following errors:

1. **Missing closing tags**: The `<port>` tag does not have a closing tag, and there is a missing closing angle bracket `>` in the `<rateLimit>` tag.
2. **Incorrect escaping of special characters**: In the user name `John  & Doe`, the ampersand `&` is not escaped.

### Solutions
1. I added the missing closing tags and corrected the missing angle bracket:
   - Added `</port>` to close the `<port>` tag.
   - Added `>` to close the `<rateLimit>` tag correctly.

The corrected part should look like this:
```xml
<port>3306</port>
```
```xml
<rateLimit>1000</rateLimit>
```

2. I corrected the special character in the user name. The proper escape for the ampersand is `&amp;`.

The corrected user entry should look like this:
```xml
<name>John &amp; Doe</name>
```

## Task 2:
You need to add a new configuration setting to the XML file. The new setting is a
boolean value called enableDebugMode. Add this setting to the appropriate section of the XML
file, setting it to true. Explain where you added it and why.

I will add a new boolean configuration setting called `enableDebugMode` and set it to `true`. This setting is relevant for developers to allow debugging within the application. The most appropriate section to add this setting is within the `<settings>` section, as it pertains to the operation configuration of the application. I will place it above the `<logging>` section.

### Revised XML Snippet

Here's how the revised XML file segment will look after adding the new setting:

```xml
<settings>
  ...
  <enableDebugMode>true</enableDebugMode>
  <logging>
    <level>DEBUG</level>
    <file>/var/log/myapp.log</file>
  </logging>
  ...
</settings>
```


## Task 3:
You are given a small web application (provided – this could be a simple
HTML/CSS/JS application or a very basic PHP application). This application needs to be
deployed to a staging server. Describe the steps you would take to deploy this application, considering best practices for scalability, security, and maintainability

To deploy a small web application (HTML/CSS/JavaScript) to a staging server, I would follow these steps:

### Deploying a Web Application (JavaScript) to Firebase Hosting

#### 1. **Clone the Existing Git Repository**
   - Open a terminal and navigate to the desired project location.
   - Use the following command to clone the existing repository:

     ```bash
     git clone <repository-url>
     ```
   - Navigate into the cloned project directory:
     ```bash
     cd <project-name>  # Replace <project-name> with the name of the cloned project
     ```

#### 2. **Setting Up Firebase Hosting**

##### **Set Up Firebase Project:**
   - Go to the [Firebase Console](https://console.firebase.google.com/) and create a new Firebase project.
   - Click on "Add project" and follow the prompts to fill in the necessary details.

##### **Install Firebase CLI:**
   - Install the Firebase CLI globally using npm:

     ```bash
     npm install -g firebase-tools
     ```

##### **Initialize Firebase Hosting:**
   - In the project directory, run the following command and follow the prompts:

     ```bash
     firebase init hosting
     ```
   - Choose the project created and configure the public directory (this is typically `build` or `public` based on the structure).
   - When prompted, configure it as a single-page app if applicable, and confirm if you want to overwrite `index.html`.

##### **Create Firebase Configuration File:**
   - The Firebase configuration will be in a `firebase.json` file created during the initial setup. Make sure this aligns with the hosting structure.

#### 3. **Implement Best Practices & Documentation**

- **Scalability**:
  - **Static Assets Management**: Utilize Firebase's caching capabilities and set appropriate caching headers in the `firebase.json` to allow static assets to be cached for optimal performance.
  - **Content Delivery Network (CDN)**: Firebase Hosting automatically serves the apps from a global CDN, reducing latency and increasing speed.
- Documentation on [Static Assets Management](https://firebase.google.com/docs/hosting/manage-cache)
and
[Content Delivery Network (CDN)](https://cloud.google.com/cdn?hl=en)


- **Security**:
  - **Firebase Security Rules**: Use Firebase's security rules to control access to the apps' data, ensuring sensitive information is protected.
  - **HTTPS Enforcement**: Firebase Hosting automatically provisions SSL certificates for the apps' domains, ensuring that data is sent over HTTPS by default.
- Documentation on [Firebase Security Rules](https://firebase.google.com/docs/rules)
and
[HTTPS Enforcement](https://firebase.google.com/docs/hosting/test-preview-deploy#serving_over_https)

- **Maintainability**:
  - **Logging and Monitoring**: Use Firebase Analytics and Crashlytics to monitor app performance and capture error logs, helping diagnose issues proactively.
  - **Configuration Management**: Store sensitive environment variables and API keys separately, using methods like Firebase Functions or using GitHub Secrets to avoid exposing sensitive data in your codebase.
- Documentation on [Logging and Monitoring](https://firebase.google.com/docs/functions/writing-and-viewing-logs?gen=2nd)
and
[Configuration Management](https://firebase.google.com/docs/functions/config-env?gen=2nd)

- **Documentation**:
  - Document the deployment process, specific configurations, instructions for local setup, and usage guidelines directly in the repository.
  - **If the GitHub repository has Wiki enabled**, create comprehensive documentation there.
  - **Alternatively**, update the `README.md` file in the root of the project to include:
    - Detailed setup instructions
    - Build and deployment processes
    - Environment variable configurations
    - Testing procedures and how to contribute
    - Troubleshooting tips and common issues

#### 4. **GitHub Workflow for Development Branch with PR Preview**

##### **Set Up GitHub Actions:**
   - In the repository, create a folder named `.github/workflows/`.
   - Create a new file named `preview-deployment.yml`.

##### **Add Workflow for Development Preview:**
   - Add the following YAML configuration to deploy to Firebase Hosting when a pull request is made against the `development` branch:

   ```yaml
   name: Deploy Preview to Firebase Hosting

   on:
     pull_request:
       branches:
         - development

   jobs:
     deploy-preview:
       runs-on: ubuntu-latest

       steps:
         - name: Checkout code
           uses: actions/checkout@v2

         - name: Set up Node.js
           uses: actions/setup-node@v2
           with:
             node-version: '14' # Specify the Node.js version

         - name: Install dependencies
           run: npm install

         - name: Build the application
           run: npm run build # Change to the build command

         - name: Firebase Deploy
           env:
             FIREBASE_TOKEN: ${{ secrets.FIREBASE_TOKEN }} # Store the token in GitHub secrets
           run: |
             firebase deploy --only hosting --token $FIREBASE_TOKEN --project <THE_FIREBASE_PROJECT_ID>
   ```

##### **Generate Firebase Token:**
   - Run the following command to generate a token:

     ```bash
     firebase login:ci
     ```
   - Save the generated token in the GitHub repository secrets as `FIREBASE_TOKEN`.

#### 5. **GitHub Workflow for Staging Branch after Approval**

##### **Create Staging Deployment Workflow:**
   - Create another file in the same `.github/workflows/` directory called `staging-deployment.yml`.

##### **Add Workflow Configuration for Staging Deployment:**
   - Here’s a sample YAML configuration for deploying to Firebase when changes to the `development` branch are merged into the `staging` branch:

   ```yaml
   name: Deploy Staging to Firebase Hosting

   on:
     push:
       branches:
         - staging

   jobs:
     deploy-staging:
       runs-on: ubuntu-latest

       steps:
         - name: Checkout code
           uses: actions/checkout@v2

         - name: Set up Node.js
           uses: actions/setup-node@v2
           with:
             node-version: '14' # Specify the Node.js version

         - name: Install dependencies
           run: npm install

         - name: Build the application
           run: npm run build # Change to the build command

         - name: Firebase Deploy
           env:
             FIREBASE_TOKEN: ${{ secrets.FIREBASE_TOKEN }} # Store the token in GitHub secrets
           run: |
             firebase deploy --only hosting --token $FIREBASE_TOKEN --project <THE_FIREBASE_PROJECT_ID>
   ```

#### 6. **Summary of Workflows**

- **Preview Workflow**: This workflow runs when a pull request is made against the `development` branch. It installs dependencies, builds the project, and deploys to a preview URL provided by Firebase for that pull request.

- **Staging Workflow**: This workflow runs when there is a push to the `staging` branch after the `development` branch has been approved. It follows similar steps: checking out the code, installing dependencies, and building the project before deploying.

#### 7. **Testing the Workflow Setup**

- Create a feature branch and push changes to trigger the preview workflow.
- Create a pull request to `development` to test the preview deployment.
- Once the changes are reviewed and approved in `development`, merge them into `staging` to test the staging workflow.
