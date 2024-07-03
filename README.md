-
##User and group Management  Script by Maurice Makafui
--

##The Overview
--
On a Linux system, the `create_users.sh` script is intended to automate the process of creating and managing user accounts. The script simplifies user provisioning with an emphasis on efficiency and security by reading an input file that contains usernames and related groups. System administrators and DevOps engineers who are in charge of handling user accounts in dynamic, large-scale settings may find this tool very helpful.

##Features
--

**Automated User Creation**: This process uses data from a text file to create users and their principal groups.

**Create safe, unique passwords for every user with the help of random password generationGroup Management** : Assigns users to designated groups and establishes new ones if none already exist.

**Detailed Logging**: Records every activity for audit and troubleshooting purposes in `/var/log/user_management.log` .
**Safe Password Storage**: Restricted access passwords are stored safely in`/var/secure/user_passwords.csv`.
.

##Requirements needed for successfull task.
--

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
--
check the example below:

Maurice1;staging,development,deployment

Gwenny;prayergroup

Felix;fitness,gymgroup
--
Maurice1,Gwenny and felix
Staging, dev, and www-data are the group names.


Run this command to output all current human users.
--
<awk -F: '$3 >= 1000 {print $1}' /etc/passwd>

Note: Do well to look out for the groups in your input text file
3. Run the script with the following command:
Note: You could also output all existing users with

<cat /etc/passwd>

 but it will output all users, not just human users.

Run the id command for a specific user
--
id <username>

sample case.

id Felix
If the user exists, this will display information about the user. If not, it will show an error message.

Run the following command to view and verify all the home directories of the created users

cd /home && ls

Run this to output all groups
--
cat /etc/group

Run this to check the existence of specific groups
--
getent group <groupname>

eg.

getent group dev

or

getent group sudo

Once you run it with the specific group name, it will show you the group (if it exists)
and the users assigned to it. If not, it will show no output.

Run this to output content of log file
--
cat /var/log/user_management.log

Run this command to verify the access permissions on /var/log/user_management.log

ls -al /var/log/user_management.log

Run this command to view passwords, verify if user and password are delimited by "," and passwords are hashed

cat /var/secure/user_passwords.csv

Run to output the content and verify the access permissions on /var/secure/user_passwords.csv
ls -al /var/secure/user_passwords.csv


Script Details


## `create_users.sh`

```
#!/bin/bash

# Log file location
LOGFILE="/var/log/user_management.log"
PASSWORD_FILE="/var/secure/user_passwords.csv"

# Ensure the script is run as root
if [[ "$EUID" -ne 0 ]]; then
  echo "Please run as root"
  exit 1
fi

# Check if the input file is provided
if [ -z "$1" ]; then
  echo "Error: No file was provided"
  echo "Usage: $0 <name-of-text-file>"
  exit 1
fi

# Create log and password files
mkdir -p /var/secure
touch "$LOGFILE" "$PASSWORD_FILE"
chmod 600 "$PASSWORD_FILE"

# Function to generate a random password
generate_random_password() {
    local length=${1:-12} # Default length is 12 if no argument is provided
    LC_ALL=C tr -dc 'A-Za-z0-9!?%+=' < /dev/urandom | head -c $length
}

# Function to create a user
create_user() {
  local username=$1
  local groups=$2

  if getent passwd "$username" > /dev/null; then
    echo "User $username already exists" | tee -a "$LOGFILE"
  else
    useradd -m -g "$username" -s /bin/bash "$username"
    echo "Created user $username" | tee -a "$LOGFILE"
  fi

  # Create the user's personal group
  if ! getent group "$username" > /dev/null; then
    groupadd "$username"
    echo "Created group $username" | tee -a "$LOGFILE"
  fi

  # Add user to specified groups
  IFS=',' read -r -a groups_array <<< "$groups"
  for group in "${groups_array[@]}"; do
    if ! getent group "$group" >/dev/null; then
      groupadd "$group"
      echo "Created group $group" | tee -a "$LOGFILE"
    fi
    usermod -aG "$group" "$username"
    echo "Added user $username to group $group" | tee -a "$LOGFILE"
  done


# Set up home directory permissions
  chmod 700 /home/"$username"
  chown "$username:$username" /home/"$username"
  echo "Set up home directory for user $username" | tee -a "$LOGFILE"

  # Generate a random password
  password=$(generate_random_password)
  echo "$username:$password" | chpasswd
  echo "$username,$password" >> "$PASSWORD_FILE"
  echo "Set password for user $username" | tee -a "$LOGFILE"
}

# Read the input file and create users
while IFS=';' read -r username groups; do
  # Skip empty lines
  if [[ -z "$username" ]]; then
    continue
  fi
  create_user "$username" "$groups"
done < "$1"

echo "User creation process completed." | tee -a "$LOGFILE"

```



My nternship Journey with HNG so far
The creation of "create_users.sh" is evidence of the priceless knowledge I acquired throughout my HNG internship. This rigorous curriculum gave me the tools I needed to hone my DevOps abilities, face practical problems, and come up with workable answers. Having the chance to work on dynamic, large-scale projects has strengthened my automation and system administration skills.


Explore the HNG Internship [here](https://hng.tech/internship) and discover how it can propel your career to new heights. If you're looking to hire talented tech professionals, check out  also [HNG Hire](https://hng.tech/hire).

##Contributing
Contributions are all accepted! Feel free to fork the repository, make improvements, and submit a pull request.

##License
This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.


