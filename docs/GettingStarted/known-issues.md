## Known Issues

!!! bug "IP Configuration"
    **The Issue**

    - When sharing your project, the `Public IP` field may show incorrect values
    - This happens because IP addresses are stored in the scene file
    
    **How to Fix**

    1. Select the `Network Manager` in your scene
    2. Right-click on the component
    3. Choose `Force Get Public IP`
    
    **Note**

    - Your correct IP will be automatically fetched
    - This step is required for proper network connections
    - Remember to update IP when changing networks

!!! bug "Installation Troubleshooting"
    **Common Issues**

    - Unity freezing during installation
    - Missing macro definitions
    - Package import errors
    
    **How to Fix**

    1. Show hidden files in Windows:
        - Open File Explorer
        - View > Show > Hidden items
    
    2. Delete macro file:
        - Go to: `Assets/Plugins/OmniNetworking`
        - Find: `omni_macros` file
        - Delete it
    
    3. Restart Unity:
        - Close Unity completely
        - Reopen your project
        - Package will regenerate macros

!!! bug "Package Update Issues"
    **The Issue**

    - After updating Omni via Package Manager, you may encounter:
        - Missing script references
        - Unknown script errors
        - Missing component warnings
    
    **How to Fix**

    1. Remove Omni package:
        - Open Package Manager
        - Select Omni Networking
        - Click "Remove"
    
    2. Restart Unity:
        - Close Unity completely
        - Reopen your project
    
    3. Reinstall Omni:
        - Open Package Manager
        - Install Omni Networking
        - Wait for import completion

    **Note**
    
    - This process will resolve reference issues caused by package updates
    - Your network configurations will be preserved

---