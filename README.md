# FullStack-IIS-Deployment

Script from Claude for FULL STACK WEBSITE DEPLOYMENT ON IIS


Last time we only hosted the json.
Now we have to deploy the frontend as well as the backend code on IIS
So tell me the steps for the same

What stack is the backend built with? IIS setup differs a lot depending on whether it's .NET, Node.js, or something else (Python/Java, etc.), so I want to give you the right steps rather than a generic guess.

What's the backend built with?
.NET (ASP.NET Core / Framework)
What's the frontend built with?
React/Angular/Vue (SPA, static build)

Here's the full setup for deploying an ASP.NET Core backend + React SPA frontend on IIS, assuming they'll live as two separate sites/apps (typical setup — API on one path/port, static React build served separately or under the same site).

1. Prerequisites on the IIS server
Install the IIS role (Web Server (IIS) via Server Manager, or Install-WindowsFeature -Name Web-Server in PowerShell)
Install the .NET Hosting Bundle (not just the runtime) — this includes the ASP.NET Core Module (ANCM) that lets IIS talk to Kestrel. Download from Microsoft's .NET download page, matching your app's .NET version.
After installing the hosting bundle, run iisreset to make sure IIS picks up the module.
Verify: open IIS Manager → check that "ASP.NET Core Module" appears under modules, or check %windir%\System32\inetsrv\config\applicationHost.config for the module entry.
2. Publish the backend (.NET)

On your dev machine:

dotnet publish -c Release -o ./publish

This gives you a folder with the compiled DLLs, web.config (auto-generated), and appsettings.json.

3. Deploy backend to IIS
Copy the ./publish folder contents to a folder on the server, e.g. C:\inetpub\wwwroot\myapi
In IIS Manager:
Create a new Application Pool (e.g. MyApiPool) → set .NET CLR version = No Managed Code (Kestrel handles the runtime, not IIS's managed pipeline)
Create a new Site (or Application under an existing site) pointing to that folder, bind it to a port or hostname
Assign it to MyApiPool
Check web.config in the publish folder has the right processPath/arguments pointing to your app DLL — this is usually auto-generated correctly by dotnet publish.
Set folder permissions: give the app pool identity (IIS AppPool\MyApiPool) read/execute rights on the folder (and write rights if it logs/writes files).
Update appsettings.json/appsettings.Production.json with production connection strings, CORS origins (must include your frontend's URL), etc.
4. Build the frontend (React/Angular/Vue)
npm run build

This produces a static build (or dist) folder — plain HTML/CSS/JS, no Node needed on the server.

5. Deploy frontend to IIS
Copy the build output to e.g. C:\inetpub\wwwroot\myfrontend
Create another Site (or virtual directory/application) in IIS pointing to that folder
Static files don't need a .NET app pool with CLR — a plain app pool works fine (can even reuse the default one)
Important for SPAs: add URL rewriting so client-side routes (e.g. /dashboard) don't 404 on refresh. Install URL Rewrite Module for IIS, then add a web.config in the frontend folder:
xml
<configuration>
  <system.webServer>
    <rewrite>
      <rules>
        <rule name="React Routes" stopProcessing="true">
          <match url=".*" />
          <conditions logicalGrouping="MatchAll">
            <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
            <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
          </conditions>
          <action type="Rewrite" url="/index.html" />
        </rule>
      </rules>
    </rewrite>
  </system.webServer>
</configuration>
6. Connect frontend to backend
Make sure your React app's API base URL (env var, e.g. REACT_APP_API_URL) points to the deployed backend's URL/port, and rebuild before deploying if you change it — env vars get baked in at build time for CRA/Vite.
On the backend, configure CORS in Program.cs/Startup.cs to allow the frontend's origin.
7. Bindings, SSL, firewall
Set proper hostname/port bindings for both sites in IIS
If using HTTPS, bind an SSL certificate to each site
Open the relevant ports in Windows Firewall if external traffic needs access
8. Test
Hit the backend directly (e.g. https://yourdomain/api/health) to confirm it's running
Load the frontend, confirm API calls succeed (check browser console/network tab for CORS or 404 errors)
Refresh a deep frontend route to confirm the rewrite rule works

If you want, tell me whether frontend and backend are going under the same site (e.g. API under /api subpath) vs separate sites/ports — the IIS structure changes a bit depending on that, and I can give you the exact config either way.

dotnet publish -c Release -o ./publish

Instead of this :-

dotnet publish -c Release -o ./publish

we can click on publish also?

Yes, absolutely — if you're using Visual Studio, right-clicking the project and choosing Publish does the same thing through a GUI, and it's actually more convenient for IIS deployments since it can automate more of the process.

Here's how that works:

Using Visual Studio's Publish GUI
Right-click the project in Solution Explorer → Publish
Choose a publish target:
Folder — simplest option, builds and drops files into a local folder (same result as the CLI command), which you then manually copy to the IIS server
Web Deploy (IIS, FTP, etc.) — if the IIS server has the Web Deploy feature/agent installed, Visual Studio can publish directly to the server over the network, no manual copying needed
Configure the profile:
Set Configuration to Release
If using Folder, set the target path
If using Web Deploy, enter the server address, site name, and credentials
Click Publish — VS builds, publishes, and (with Web Deploy) pushes the files straight to the IIS site
Folder publish vs dotnet publish CLI

They're functionally identical — the Folder publish option in VS just runs dotnet publish under the hood with the settings from your .pubxml profile. Same output: DLLs, web.config, appsettings.json in the target folder. Use whichever you're more comfortable with; CLI is easier to script/automate (CI/CD), GUI is easier for a one-off manual deploy.

Web Deploy option (if available)

This is the closest thing to a true one-click deploy to IIS:

Requires the Web Deploy component installed on the IIS server (Web Platform Installer or standalone installer)
Requires the Web Management Service running on the server with remote access enabled
Visual Studio then pushes files directly over HTTPS to the IIS site, and can even create the app pool/site config for you if permissions allow

If you don't already have Web Deploy set up on the server and just want to get this done quickly, the Folder publish + manual copy (or your existing FTP/RDP process) is the least setup-heavy — that's likely what fits your "we only hosted the JSON last time" workflow so far.

No need to click into it right now — just seeing the icon there confirms the module installed successfully.

You can leave it as is (the web.config rule you already added is already active automatically now that the module exists — you don't need to configure anything through this UI).

Just go test your site now
Go to your browser
Navigate to http://localhost:8081
Hard refresh: Ctrl+Shift+R

The 500.19 error should be gone, and your Employee Management System should load normally. Try navigating to an inner route (if your app has one, like clicking into an employee or an add/edit page) and then refresh the browser on that page — that's the real test of whether the rewrite rule is working (confirms deep-link refreshes don't 404 anymore).

Let me know what you see.
