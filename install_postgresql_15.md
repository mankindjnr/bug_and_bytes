It seems that PostgreSQL client version 15 might not be available in the default repositories for Linux Mint. Here's an alternative approach to install the PostgreSQL client:

Add PostgreSQL Apt Repository: PostgreSQL provides its own repository for different versions. You can add this repository to install version 15.

First, download and add the GPG key used for signing packages:

```bash
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
```

Add PostgreSQL Repository: Add the PostgreSQL repository for Linux Mint 20 (replace mint with your Linux Mint version codename if different):

```bash
echo "deb http://apt.postgresql.org/pub/repos/apt/ $(lsb_release -cs)-pgdg main" | sudo tee /etc/apt/sources.list.d/pgdg.list
```

Update Package List: Update your package list to include the PostgreSQL packages from the new repository:

```bash
sudo apt update
```

Install PostgreSQL Client: Now you can install the PostgreSQL client version 15:

```bash
sudo apt install postgresql-client-15
```

Verify Installation: Check the PostgreSQL client version to confirm it's installed correctly:

```bash
psql --version
```
