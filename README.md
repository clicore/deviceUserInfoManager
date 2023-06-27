# Device UserInfo Manager
This project was created to manage meshagent information, access to user applications and files on a connected device and auto update it on the Meshcentral server for display.

Admin can set up the OS with all new settings and install applications, and then connect to Meshcentral Server. 
This custom image can now be installed by various users on their devices. Upon first boot, in either CLI or GUI mode 
users will be forced to type in correct credentials, after which users will be allowed access to CLI or GUI as usual. 
This user information will then be carried over to the Meshcentral Server account, to identify users on each device. 
Users can modify user information after first boot via running a CLI command.

All project files and doc refer to installing and operating on a connected Linux x86-64 device, 
for both CLI headless mode and GUI desktop mode.

Tested on Debian 11 and DietPi with LXDE graphical environment.

### Screenshots
![1](https://github.com/Hiliaka-MobiTV/DeviceUserInfoManager/assets/134340514/258822b7-b104-4af3-b7c7-0529e32edc4a)
![2](https://github.com/Hiliaka-MobiTV/DeviceUserInfoManager/assets/134340514/62fe7e65-1796-489b-aa91-7fb644b95926)
![3](https://github.com/Hiliaka-MobiTV/DeviceUserInfoManager/assets/134340514/84b07170-eb17-4168-be8d-f2dd7b50fb52)


### Architecture
Divided into 2 parts:
1. User side - provides input interfaces.
2. Admin (root) side - provides the meshagent.tag management, commits all changes and overwrites and controls an access
to the selected files or directories.

User side files:
1. userinfo_cli.py - CLI input interface for changing device user information.
2. userinfo_gui.py - GUI input interface for changing device user information.

Admin (root) side files:
1. agent_manager/config.py - configuration file.
2. agent_manager/agent_manager.py - needed to manage the meshagent.tag file.
3. agent_manager/agent_service.py - needed to check and commit all changes to the meshagent.tag.
4. agent_manager/boot.py - shows CLI/GUI input interfaces if the agent tag does not exist on system boot.

### How it works
**Admin (root) side** are Python scripts that will work from root side and execute 
such actions as: 
1. Changing an agent tag of the meshagent.tag file.
2. Access control to user programs.
3. Shows CLI/GUI input interfaces if an agent tag does not exist.

Simple users do not have any access to admin side files.
Only admins can configure the config.py file and other features of admin side.
Admins can connect to user device via MeshCentral admin panel/SSH and manage user's system.
Files are entered in ACCESS_CONTROL_PATHS will always be restricted and files are entered in WHITELIST_PATHS will
always be allowed. Parameters ACCESS_CONTROL_PATHS and WHITELIST_PATHS are configurable in the config.py.

**Warning!** To allow the files or applications that was entered to ACCESS_CONTROL_PATHS and restricted, admins needs to remove these paths from ACCESS_CONTROL_PATHS and enter these paths in WHITELIST_PATHS or change permissions manually. Otherwise, admin (root) side will always restrict the applications or files that in ACCESS_CONTROL_PATHS.

**User (non-root) side** are Python scripts that will be allowed from user account of the system. 
These scripts can be running by users that do not have a root access manually.
User side contains two Python scripts that provide CLI and GUI input interfaces. In these interfaces users will enter
their names and phone numbers. When users have entered their names and phone numbers one of this interface script will
make the deviceUserInfo file where will be written this data; also auto retrieving the device LAN MAC ID.

After this, admin (root) side script will get this data and put in the ~/meshagent/meshagent.tag (root permissions only) in last row like an agent tag.
Users cannot close GUI input interface while the entry fields are empty or entered incorrectly.

On admin (root) side has a systemd service (agent_service.py) that will work all time and compare the meshagent.tag and the deviceUserInfo file. The meshagent.tag file will be overwritten if the deviceUserInfo contains a different agent tag.

Every time on system boot will be run the boot.py script. This script will run scripts that will show CLI/GUI input interfaces for users if an agent tag in the meshagent.tag file does not exist. CLI and GUI interfaces will not pop up on first system boot if the meshagent.tag has an agent tag information. Accordingly, an agent tag will be empty in the meshagent.tag file on first system boot!

### Configuration file
On the admin (root) side, the agent_manager service has simple configuration options. These options are listed below:
1. BASE_AGENT_DIR - directory where the meshagent is located.
2. DEVICE_USERINFO_NAME - user information file name, will be compared with the information 
of meshagent.tag file when user enters data.
3. ACCESS_CONTROL_PATHS - list of paths(directories or programs) where an access will be restricted.
4. WHITELIST_PATHS - list of paths that do not require access control.


We will restrict certain applications or files, 
but will still allow other packages that are required for basic OS functioning.
Access control example:

```python
# Paths to directories or files that will be controllable
# Example: Path('/home/user/programs')
ACCESS_CONTROL_PATHS = [
    Path("/home/user"), # deny access to all files/applications in this directory
    Path("/usr/bin/kodi"), # restricts Kodi
    Path("/usr/bin/vlc"), # restricts VLC
    Path("/opt/google/chrome/google-chrome") # restricts Google Chrome
]

# Paths to programs that do not need to be denied access
# Example: Path('/home/user/programs/')
WHITELIST_PATHS = [
    Path("/home/user/userinfo_cli.py"),
    Path("/home/user/userinfo_gui.py")
]
```

### Auto installation guide
This auto installation script will install all the necessary files(user side and admin side) to their directories.
User side scripts will be extracted to user's home folders such as /home/username/.

1. Make sure the device is connected to any MeshCentral server.
2. Open the config.py and configure it.
3. Run the install.py script with root permissions.

### Manual installation guide
Make sure you copy/create/edit all the files with correct permissions(for admins and users) when you install these scripts manually.
Otherwise, it will not work correctly!

1. Make sure the device is connected to any MeshCentral server.
2. Copy the agent_manager folder to /usr/local/mesh_services/meshagent/agent_manager and configure the config.py file.
3. Copy the agent_manager.service file to /etc/systemd/system/agent_manager.service 
or create your own .service file with params like in the agent_manager.service.
4. Run the following commands with root permissions:
```commandline
systemctl daemon-reload
systemctl enable agent_manager
systemctl start agent_manager
```

5. To pop up the CLI/GUI input interfaces on first system boot it necessary to create/edit such 
files as .bashrc, .bash_profile, .profile, .xsessionrc.
You can edit these files selectively, depending on your requirements. These files must be owned by user and have user permissions, it is important!

Descriptions:<br>
.bashrc - will run on terminal instance startup.<br>
.bash_profile - will run on user login his account without a graphical environment.<br>
.profile - will run on user login his account using a graphical environment(Debian).<br>
.xsessionrc - will run on user login his account using a graphical environment(X11, DietPi).<br>

Create/edit .bash_profile file, add this code:
```shell
if [[ -f ~/.bashrc ]]; then
	. ~/.bashrc
fi
```
Note: This code will start .bashrc file with all of its scripts on user login his account!

Edit .bashrc, .profile file and .xsessionrc, add this line:

```shell
/usr/bin/python3 /usr/local/mesh_services/meshagent/agent_manager/boot.py
```
Copy userinfo_cli.py and userinfo_gui.py files to the user's home directory, otherwise boot.py will not find them.

### Usage

1. Upon first device boot, the user will be asked to input credentials (Name and Phone Number.) 

In CLI (headless) mode, there will be command prompts, first for Name and then a prompt for Name (confirmation); then subsequent prompts for Phone Number and Phone Number (confirmation.)

Similarly in GUI (desktop) mode, there will be a pop-up dialog box where the user will type in fields for Name, Name (confirmation), Phone Number, and Phone Number (confirmation.)

Also note that the userinfo script will auto-retrieve the device LAN MAC ID and send it along with Name And Phone Number user input to the meshagent.tag file.

User will not be able to access the CLI or desktop screen unless all fields are typed in with confirmation.
Once user information is correctly entered, subsequent boots will not show any CLI prompts or GUI dialog box, respectively. User can continue with CLI commands or desktop clicks as usual.

2. In addition, in case updating user information is needed, after first device boot:

Run userinfo_cli.py or userinfo_gui.py script to open input interface, enter your name and phone number.
After this, the deviceUserInfo file will be created in the /home/username folder,
the agent service will check this file and rewrite if there are changes. All these files must reside on the device, not on Meshcentral server.

To run CLI use the command:
```commandline
python3 userinfo_cli.py
```
To run GUI use the command:
```commandline
python3 userinfo_gui.py
```
Note:
A meshcentral administrator can log in using SSH and change the meshagent.tag file manually. 
However, make sure the deviceUserInfo file does not exist because the agent_manager service will 
compare an agent tag that in the meshagent.tag file with the deviceUserInfo and overwrite this information.
