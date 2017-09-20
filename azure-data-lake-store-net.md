# Get started with Azure Data Lake Store using .NET SDK

## Prerequisites
* **Visual Studio 2015, or 2017**.

* **An Azure subscription**. See [Get Azure free trial](https://azure.microsoft.com/pricing/free-trial/).

* **Azure Data Lake Store account**. For instructions on how to create an account, see [Get started with Azure Data Lake Store](https://docs.microsoft.com/en-us/azure/data-lake-store/data-lake-store-get-started-portal)

* **Azure Active Directory Application**. In this tutorial we use an Azure AD application client secret to retrieve an Azure Active Directory token (service-to-service authentication). We use this token to create a Data Lake Store client object to perform operations file and directory operations. For instructions on how to authenticate with Azure Data Lake Store using the client secret, we perform the following high-level steps:
                                          
    1. Create an Azure AD web application.
    2. Retrieve the client ID, client secret, and token endpoint for the Azure AD web application.
    3. Configure access for the Azure AD web application on the Data Lake Store file/folder that you want to access from the .NET application you are creating.
   
    For instructions on how to perform these steps, see [Create an Active Directory application](https://docs.microsoft.com/en-us/azure/data-lake-store/data-lake-store-authenticate-using-active-directory).
    
    Azure Active Directory provides other options as well to retrieve a token. You can pick from a number of different authentication mechanisms to suit your scenario, for example, an application running in a browser, an application distributed as a desktop application, or a server application running on-premises or in an Azure virtual machine. You can also pick from different types of credentials like passwords, certificates, 2-factor authentication, etc. In addition, Azure Active Directory allows you to synchronize your on-premises Active Directory users with the cloud. For details, see [Authentication Scenarios for Azure Active Directory](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-authentication-scenarios). 


## Create a .NET application

The code sample available [on GitHub](https://github.com/azure-samples/data-lake-store-adls-dot-net-get-started/) walks you through the process of creating files in the store, downloading a file, renaming files and deleting some files in the store.

Create a Console Application (for this tutorial we create .NET framework 4.5.2). For instructions on how to create a .NET project using Visual Studio, see [here](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/inside-a-program/hello-world-your-first-program).

## Add Nuget dependencies
   1. Right-click the project name in the Solution Explorer and click **Manage NuGet Packages**.
   2. In the **Nuget Package Manager** window make sure that **Package source** is set to **nuget.org** and that **Include prerelease** check box is selected. 
   4. Under **Browse** search **adls**. Select and Install `Microsoft.Azure.DataLake.Store`. It will install all other necessary packages.

## Add the application code
This section of the article walk you through the main parts of the code. There are three main parts to the code.

1. Obtain the Azure Active Directory token
2. Use the token to create a Data Lake Store client.
3. Use the Data Lake Store client to perform operations.

#### Step 1: Obtain an Azure Active Directory token.
The following code is creating service-to-service authentication with client secret non-interactively. However the SDK does not mandate to use this method only. You can create other methods like service to service authentication with certificate.
 
Replace **FILL-IN-HERE** with the actual values for the Azure Active Directory Web application.

    private static string clientId = "FILL-IN-HERE";         // Also called application id in portal
    private static string clientSecret = "FILL-IN-HERE";
    private static string domain = "FILL-IN-HERE";            // Also called tenant Id
            
    var creds = new ClientCredential(clientId, clientSecret);
    var clientCreds = ApplicationTokenProvider.LoginSilentAsync(domain, creds).GetAwaiter().GetResult();

#### Step 2: Create an Azure Data Lake Store client (AdlsClient) object
Creating an AdlsClient object requires you to specify the Data Lake Store account name and the token provider you generated in the last step. Note that the Data Lake Store account name needs to be a fully qualified domain name. For example, replace **FILL-IN-HERE** with something like **mydatalakestore.azuredatalakestore.net**.

    private static string clientAccountPath = "FILL-IN-HERE";
    // Create ADLS client object
    AdlsClient client = AdlsClient.CreateClient(clientAccountPath, clientCreds);

#### Step 3: Use the AdlsClient to perform file and directory operations
The code below contains example snippets of some common operations.

Note that files are read from and written into using standard .NET streams. This means that you can layer any of the .NET streams on top of the Data Lake Store streams to benefit from standard .NET functionality.

#####Create file

    string fileName = "/Test/testFilename1.txt";
        
    // Create a file - automatically creates any parent directories that don't exist
    using (var streamWriter = new StreamWriter(client.CreateFile(fileName, IfExists.Overwrite)))
    {
        streamWriter.WriteLine("This is test data to write");
        streamWriter.WriteLine("This is line 2");
    }

#####Append existing file

     // Append to existing file
     using (var streamWriter = new StreamWriter(client.GetAppendStream(fileName)))
     {
         streamWriter.WriteLine("This is the added line");
     }

#####Read file

     //Read file contents
     using (var readStream = new StreamReader(client.GetReadStream(fileName)))
     {
         string line;
         while ((line = readStream.ReadLine()) != null)
         {
             Console.WriteLine(line);
         }
     }

#####Retrieve File/Directory properties

     // Get the properties of the file
     var directoryEntry = client.GetDirectoryEntry(fileName);
     PrintDirectoryEntry(directoryEntry);


#####Rename file

     // Rename a file
     string destFilePath = "/Test/testRenameDest3.txt";
     Console.WriteLine(client.Rename(fileName, destFilePath, true));

#####Enumerate directory contents

     // Enumerate directory
     foreach (var entry in client.EnumerateDirectory("/Test"))
     {
         PrintDirectoryEntry(entry);
     }

#####Delete directory

     // Delete a dirtectory and all it's subdirectories and files
     client.DeleteRecursive("/Test");