---
##User Management Automation Script


##he Overview
On a Linux system, the `create_users.sh` script is intended to automate the process of creating and managing user accounts. The script simplifies user provisioning with an emphasis on efficiency and security by reading an input file that contains usernames and related groups. System administrators and DevOps engineers who are in charge of handling user accounts in dynamic, large-scale settings may find this tool very helpful.

##Features

**Automated User Creation**: This process uses data from a text file to create users and their principal groups.

**Create safe, unique passwords for every user with the help of random password generationGroup Management** : Assigns users to designated groups and establishes new ones if none already exist.

**Detailed Logging**: Records every activity for audit and troubleshooting purposes in `/var/log/user_management.log` .
**Safe Password Storage**: Restricted access passwords are stored safely in`/var/secure/user_passwords.csv`.
.

##Requirements needed for successfull task

Linux environment (tested on Ubuntu)
Bash shell
OpenSSL for password generation
Root or sudo privileges to execute user and group management commands
Usage
The script is designed to be run from the command line. To use it, follow these steps:

1. Clone the Repository
Clone the repository to your local machine or directly onto your Linux server:

git clone https://github.com/Maurice-Makafui/STAGE_1_HNG_11.git

cd STAGE_1_HNG_11

2. Create a text file with the usernames and groups you want to create. The file should have 

username;group1,group2,group3
check the example below:

Maurice1;staging,development,deployment

Gwenny;prayergroup

Felix;fitness,gymgroup


3. Run the script with the following command:
sudo ./create_users.sh <inputfile.txt>

Execute the script by providing the input file as an argument. Make sure to run it with sufficient privileges (e.g., as root or with sudo):

NB:Replace <input_file> with the path to your input file.

4. Verify the Output
 After completion, the script logs all actions to /var/log/user_management.log and stores generated passwords in /var/secure/user_passwords.csv. You can review these files to verify the script's execution:

```bash
cat /var/log/user_management.log
sudo cat /var/secure/user_passwords.csv
```

5. Check Created Users and Groups
To ensure users and groups have been created correctly, you can use the following commands:

- List users: `cut -d: -f1 /etc/passwd`
- List groups: `cut -d: -f1 /etc/group`


Script Details


## `create_users.sh`

```bash
#!/bin/bash

# Check if input file is provided
if [ -z "$1" ]; then
  echo "Usage: $0 <input_file>"
  exit 1
fi

INPUT_FILE="$1"
LOG_FILE="/var/log/user_management.log"
PASSWORD_FILE="/var/secure/user_passwords.csv"

# Create log file and password file directories if they don't exist
mkdir -p "$(dirname "$LOG_FILE")"
mkdir -p "$(dirname "$PASSWORD_FILE")"

# Ensure password file permissions are secure
touch "$PASSWORD_FILE"
chmod 600 "$PASSWORD_FILE"

# Function to generate random password
generate_password() {
  openssl rand -base64 12
}

# Function to log messages
log_message() {
  echo "$(date +'%Y-%m-%d %H:%M:%S') - $1" >> "$LOG_FILE"
}

# Read and process the input file
while IFS=';' read -r username groups; do
  # Trim whitespace
  username=$(echo "$username" | xargs)
  groups=$(echo "$groups" | xargs)

  # Create user and primary group
  if id "$username" &>/dev/null; then
    log_message "User $username already exists"
  else
    useradd -m -G "$username" -s /bin/bash "$username"
    log_message "User $username created"

    # Generate and set password
    password=$(generate_password)
    echo "$username:$password" | chpasswd
    echo "$username,$password" >> "$PASSWORD_FILE"
    log_message "Password for $username set"

    # Create additional groups
    IFS=',' read -r -a group_array <<< "$groups"
    for group in "${group_array[@]}"; do
      group=$(echo "$group" | xargs)
      if ! getent group "$group" &>/dev/null; then
        groupadd "$group"
        log_message "Group $group created"
      fi
      usermod -aG "$group" "$username"
      log_message "User $username added to group $group"
    done
  fi
done < "$INPUT_FILE"

log_message "User creation process completed"

echo "Script execution completed. Check $LOG_FILE for details."
```
my nternship Journey with HNG so far
The creation of "create_users.sh" is evidence of the priceless knowledge I acquired throughout my HNG internship. This rigorous curriculum gave me the tools I needed to hone my DevOps abilities, face practical problems, and come up with workable answers. Having the chance to work on dynamic, large-scale projects has strengthened my automation and system administration skills.


Explore the HNG Internship [here](https://hng.tech/internship) and discover how it can propel your career to new heights. If you're looking to hire talented tech professionals, check out  also [HNG Hire](https://hng.tech/hire).

##Contributing
Contributions are all accepted! Feel free to fork the repository, make improvements, and submit a pull request.

##License
This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---

