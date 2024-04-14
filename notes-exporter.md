# Console exporter/importer for the session

Due to time constraints, I'm unable to propose changes through a fork and pull request directly. However, I've developed a solution that I use personally for exporting and importing local storage data with timestamped backups for web applications. Perhaps someone might find this useful or be motivated to integrate it into the application's codebase. Below is a step-by-step guide on how to use this utility.

⚠️ The API keys are also exported, so be careful with your dump files.

## Exporting Local Storage Data

1. **Open the Browser Console**: Right-click on the webpage and select “Inspect” or use the `Ctrl+Shift+I` shortcut (Cmd+Option+I on Mac), then navigate to the "Console" tab.
   
2. **Paste the Export Function**: Copy the `exportLocalStorage` function’s code.

   ```javascript
   function exportLocalStorage() {
     let dataToSave = {};
     for (let i = 0; i < localStorage.length; i++) {
       const key = localStorage.key(i);
       const value = localStorage.getItem(key);
       dataToSave[key] = value;
     }
     const dataJSON = JSON.stringify(dataToSave);
     const now = new Date();
     const timestamp = now.toISOString().match(/(\d{2,4})-(\d{2})-(\d{2})T(\d{2}):(\d{2}):(\d{2})/);
     const formattedTimestamp = `${timestamp[1].slice(-2)}${timestamp[2]}${timestamp[3]}_${timestamp[4]}${timestamp[5]}${timestamp[6]}`;
     const fileName = `${formattedTimestamp}-localStorageBackup.json`;
     const a = document.createElement("a");
     a.href = "data:text/plain;charset=utf-8," + encodeURIComponent(dataJSON);
     a.download = fileName;
     a.id = "downloadLocalStorageBackup";
     document.body.appendChild(a);
     a.click();
     document.body.removeChild(a);
   }
   ```
   
3. **Press Enter** to run the code snippet, defining the function in the current page context.

4. **Execute the Function**: Type `exportLocalStorage();` and press Enter. This triggers the download of the `localStorageBackup.json` file containing the current state of local storage.

### Importing Local Storage Data

For importing, you first need to adjust the page to allow file uploads via a temporary input element, as browsers restrict direct file system access for security reasons.

1. **Open the Browser Console**: If not already open.

2. **Create and Invoke the Import Function**: Copy the following code snippet that creates the utility functions and invokes a temporary upload interface:

   ```javascript
   function importLocalStorage(dataJSON) {
     const dataToRestore = JSON.parse(dataJSON);
     for (const key in dataToRestore) {
       if (dataToRestore.hasOwnProperty(key)) {
         localStorage.setItem(key, dataToRestore[key]);
       }
     }
     location.reload();
   }

   function createTemporaryUploadButton() {
     const input = document.createElement("input");
     input.type = "file";
     input.onchange = e => {
       const file = e.target.files[0];
       if (file) {
         const reader = new FileReader();
         reader.onload = function(event) {
           importLocalStorage(event.target.result);
         };
         reader.readAsText(file);
       }
     };
     input.click(); // Automatically open the file dialog
   }
   ```

3. **Press Enter** to define the functions in your console.

4. **Trigger File Upload**: Execute `createTemporaryUploadButton();` to programatically click the file input and open the file selector dialog.

5. **Select Backup File**: In the dialog, choose the `localStorageBackup.json` file you previously saved.

The selected file's contents are automatically read and used to restore local storage. The page may reload (if you included `location.reload();`), applying the changes immediately.

### Copy/past part

// ##########################
// # Export part
// ##########################
function exportLocalStorage() {
  // Object to hold local storage data
  let dataToSave = {};

  // Loop through all keys in local storage
  for (let i = 0; i < localStorage.length; i++) {
    const key = localStorage.key(i);
    const value = localStorage.getItem(key);
    // Save each key-value pair in the dataToSave object
    dataToSave[key] = value;
  }

  // Convert the object to a JSON string
  const dataJSON = JSON.stringify(dataToSave);

  // Creating a timestamp for the file name
  const now = new Date();
  const timestamp = now
    .toISOString()
    .match(/(\d{2,4})-(\d{2})-(\d{2})T(\d{2}):(\d{2}):(\d{2})/);
  const formattedTimestamp = `${timestamp[1].slice(-2)}${timestamp[2]}${
    timestamp[3]
  }_${timestamp[4]}${timestamp[5]}${timestamp[6]}`;
  const fileName = `${formattedTimestamp}-localStorageBackup.json`;

  // Creating a temporary anchor element to trigger the download
  const a = document.createElement("a");
  a.href = "data:text/plain;charset=utf-8," + encodeURIComponent(dataJSON);
  a.download = fileName; // Using the formatted timestamp in the file name
  a.id = "downloadLocalStorageBackup"; // Unique ID for the anchor

  // Append the anchor to the document
  document.body.appendChild(a);
  a.click(); // Trigger the download
  document.body.removeChild(a); // Clean up by removing the element
}
exportLocalStorage();

// ##########################
// # Import part
// ##########################
function importLocalStorage(dataJSON) {
  // Parse the JSON string back into an object
  const dataToRestore = JSON.parse(dataJSON);

  // Loop through the object and save each item back into local storage
  for (const key in dataToRestore) {
    if (dataToRestore.hasOwnProperty(key)) {
      localStorage.setItem(key, dataToRestore[key]);
    }
  }
  //  Reload the page
  location.reload();
}

function createUploadButtonAndHandleFileInApp() {
  // Find the target container element by ID
  const appElement = document.getElementById("app");
  if (!appElement) {
    console.warn('Element with ID "app" not found!');
    return;
  }

  // Create file input element
  const fileInput = document.createElement("input");
  fileInput.type = "file";
  fileInput.id = "fileUploadInput";
  fileInput.style.display = "none"; // Hide the file input for styling purposes

  // Create upload button
  const uploadButton = document.createElement("button");
  uploadButton.innerText = "Upload Local Storage Backup";
  uploadButton.id = "fileUploadButton";

  // Prepend elements to the 'app' container
  // If you want both elements inside 'app', use these lines:
  appElement.insertBefore(uploadButton, appElement.firstChild);
  appElement.insertBefore(fileInput, uploadButton); // Inserts the file input before the button, making it the first child

  // Or, if only the button should be visible within the 'app', just insert the button:
  // appElement.insertBefore(uploadButton, appElement.firstChild);

  // Function to handle file reading and importing local storage data
  function handleFileUpload() {
    const file = fileInput.files[0]; // Get the selected file
    if (file) {
      const reader = new FileReader();
      reader.onload = function (e) {
        const dataJSON = e.target.result;
        // Assuming the 'importLocalStorage' function is defined
        importLocalStorage(dataJSON); // Import data into local storage
      };
      reader.readAsText(file); // Read the file as text
    }
  }

  // Event listener for the file input
  fileInput.addEventListener("change", handleFileUpload);

  // Event listener for the button click
  uploadButton.addEventListener("click", () => fileInput.click());
}

// Call this function to add the upload interface to the beginning of the 'app' element.
createUploadButtonAndHandleFileInApp();
