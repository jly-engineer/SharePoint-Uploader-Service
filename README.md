# SharePoint Uploader Service

The **SharePoint Uploader Service** is a robust, native Windows Background Service designed to automate the synchronization of local files to Microsoft SharePoint. 

It is specifically engineered to handle inputs from hardware scanners and multi-function printers. It monitors a local directory (recursively), detects new files, waits for file write operations to complete, and uploads the data to a target SharePoint Document Library via the Microsoft Graph API.

## 🚀 Key Features

*   **Real-Time Monitoring:** Uses `watchdog` to detect file creation and modification instantly.
*   **Recursive Sync:** Mirrors the local folder structure to the cloud (e.g., `Local\Sub\Scan.pdf` -> `SharePoint/Sub/Scan.pdf`).
*   **Resilient File Handling:**
    *   **Lock Detection:** Intelligently pauses upload attempts if a scanner or user still has the file open (prevents "Permission Denied" errors).
    *   **Debouncing:** Prevents duplicate upload attempts when files generate multiple modification events in quick succession.
    *   **Filtering:** Automatically ignores system junk files (e.g., `Thumbs.db`, `~$temp.doc`).
*   **Native Windows Service:** Runs automatically in the background using `pywin32` (no user login required).
*   **GUI Installer:** Includes a standalone executable for easy installation, configuration, and uninstallation.

---

## ⚙️ Prerequisites: Azure Configuration

Before installing the service, you must register an application in Microsoft Azure Entra ID (formerly Active Directory).

1.  **Register App:**
    *   Go to the [Azure Portal](https://portal.azure.com/) > **App registrations**.
    *   Click **New Registration**. Name it (e.g., "Scanner Uploader").
    *   Select **Single Tenant**.
2.  **Create Secret:**
    *   Go to **Certificates & secrets** > **New client secret**.
    *   **Copy the Value immediately.** You will need this for the installer.
3.  **API Permissions:**
    *   Go to **API Permissions** > **Add a permission** > **Microsoft Graph**.
    *   Select **Application permissions** (NOT Delegated).
    *   Search for and check: **`Files.ReadWrite.All`**.
    *   Click **Add permissions**.
    *   **CRITICAL:** Click the **"Grant admin consent for [Your Org]"** button.

### Getting the SharePoint IDs
You need the **Site ID** and **Drive (Library) ID**. The easiest way to find these is using the [Microsoft Graph Explorer](https://developer.microsoft.com/en-us/graph/graph-explorer):

1.  **Site ID:** Run `GET https://graph.microsoft.com/v1.0/sites/root` (or search by hostname).
2.  **Drive ID:** Run `GET https://graph.microsoft.com/v1.0/sites/{site-id}/drives`. Find the drive where `name` is "Documents" (or your target library) and copy the `id`.

---

## 📦 Installation & Deployment

The project builds a single file executable: `SharePointUploaderSetup.exe`.

1.  **Run Installer:**
    *   Copy `SharePointUploaderSetup.exe` to the target server/PC.
    *   Right-click and select **Run as Administrator**.
2.  **Configure:**
    *   **Tenant ID / Client ID / Client Secret:** From the Azure steps above.
    *   **SharePoint Site ID / Drive ID:** From Graph Explorer.
    *   **Monitor Folder:** The local path to watch (e.g., `C:\Scans`).
    *   **SharePoint Target Folder:** (Optional) A specific subfolder in the cloud library. Leave blank to upload to the root.
3.  **Install:**
    *   Click **Install / Update Service**.
    *   The tool will register the Windows Service (`SharePointUploaderService`) and start it immediately.

### Uninstallation
To remove the service, run `SharePointUploaderSetup.exe` as Administrator and click the red **Uninstall Service** button.

---
