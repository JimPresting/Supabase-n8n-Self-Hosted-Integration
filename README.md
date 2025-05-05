# Connecting Supabase PostgreSQL to n8n on a Self-Hosted Instance

This guide explains how to connect a Supabase PostgreSQL database to a self-hosted n8n instance by configuring firewall rules and using the correct connection string.

## Step 1: Configure Firewall Rules in Your VPC Network

To allow your n8n instance to communicate with Supabase, add a firewall rule in your VPC network settings (e.g., Google Cloud, AWS, or other cloud provider).

1. Go to your **VPC Network** settings.
![image](https://github.com/user-attachments/assets/84a9dfc3-3c7a-4e27-a96b-31b267eb4f0d)
![image](https://github.com/user-attachments/assets/601b7454-b291-43d1-8a8a-d5e8a7a845e6)


3. Add a new firewall rule with the following configuration:
   - **Name**: `allow-postgres-outbound`
   - **Direction**: Outbound
   - **Destination Filter**: IPv4 ranges
   - **Destination IPv4 Ranges**: `0.0.0.0/0` (allows all destinations; restrict to Supabase IPs for better security)
   - **Source Filter**: 
     - Select `IPv4 ranges` and specify the n8n VM's IP range, or
     - Select `None` if the connection can originate from any VM
   - **Protocols and Ports**:
     - Protocol: `TCP`
     - Port: `5432` (default PostgreSQL port)
![image](https://github.com/user-attachments/assets/7b1539e1-5697-4659-8f9d-63c65b914b4b)

4. Save the firewall rule.

## Step 2: Get the Supabase Connection String

1. Log in to your **Supabase Dashboard**.
2. Navigate to your project and click the **Connect** button in the header.
![image](https://github.com/user-attachments/assets/41da7827-bc21-4787-88d8-31749debff5d)

3. Copy the PostgreSQL connection string, which looks like:

   ```
   postgresql://postgres.uniqueaddress:[YOUR-PASSWORD]@aws-0-eu-central-1.pooler.supabase.com:6543/postgres
   ```
You can use either the Transaction Pooler or the Session Pooler but I would recommend the Transaction Pooler
   - Replace `[YOUR-PASSWORD]` with the actual password for your Supabase account (omit the square brackets) and leave the rest unchanged
   - Note: The port may vary (e.g., `6543` or `5432`). Use the port provided in the connection string.

## Step 3: Configure n8n to Connect to Supabase

1. In your self-hosted n8n instance, create a new workflow or edit an existing one.
2. Add a **PostgreSQL node** to your workflow.
3. Configure the node with the following details:
   - **Host**: `aws-0-eu-central-1.pooler.supabase.com` (from the connection string)
   - **Port**: `6543` (or the port specified in the connection string)
   - **Database**: `postgres`
   - **User**: `postgres.fkelse...` (from the connection string - your unique identifier)
   - **Password**: Your Supabase account password
   - **SSL**: Enable if required (Supabase typically requires SSL)

4. Test the connection in n8n to ensure it works.

## Security Tips (Optional)

- **Restrict Firewall Rules**: Instead of allowing `0.0.0.0/0`, use Supabase’s specific IP ranges for better security. Check Supabase’s documentation for their IP addresses.
- **Use Environment Variables**: Store your Supabase password in n8n’s environment variables instead of hardcoding it.
- **Enable SSL**: Ensure SSL is enabled in the n8n PostgreSQL node for secure communication.

## Troubleshooting

- **Connection Timeout**: Verify the firewall rule allows outbound TCP traffic on port `5432` (or the port in your connection string).
- **Invalid Credentials**: Double-check the username, password, and database name in the connection string.
- **SSL Issues**: If SSL is required but not enabled, you may see connection errors. Enable SSL in the n8n node settings.
