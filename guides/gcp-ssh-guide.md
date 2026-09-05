# SSHing into Your GCP VM

You can use the Google Cloud browser terminal or connect from your own computer. You do **not** need to install an SSH server, find the VM's IP address, or configure mDNS.

In this guide:

- **VM** means your GCP VM instance.
- **Local computer** means your personal computer.
- Commands run on the VM unless marked `[local]`.

## Option 1: Browser SSH

1. Open **Compute Engine → VM instances**.
2. Find your VM and click **SSH → Open in browser window**.

This is the simplest method and requires no local setup.

## Option 2: SSH from Your Local Terminal

The course VM has no external IP address, so local SSH must use an [Identity-Aware Proxy (IAP) tunnel](https://cloud.google.com/compute/docs/connect/ssh-using-iap).

1. Install the [Google Cloud CLI](https://cloud.google.com/sdk/docs/install) on your local computer.
2. Authenticate and select your project:

   ```bash
   [local] gcloud init
   [local] gcloud config set project PROJECT_ID
   ```

3. On the **VM instances** page, find the VM's name and zone.
4. Connect:

   ```bash
   [local] gcloud compute ssh INSTANCE_NAME --zone=ZONE --tunnel-through-iap
   ```

Replace `PROJECT_ID`, `INSTANCE_NAME`, and `ZONE` with your values.

> Do not install `openssh-server`, assign an external IP, or open SSH to the internet. GCP already prepares the VM for SSH, and IAP provides the private connection.

## Set Up GitHub Access from the VM

Create a separate SSH key on the VM. Do not copy your private key from your local computer or enable SSH agent forwarding.

1. Generate a key:

   ```bash
   ssh-keygen -t ed25519 -C "YOUR_EMAIL"
   ```

   Press Enter to accept the default filename. You may add a passphrase or leave it empty.

2. Display the public key:

   ```bash
   cat ~/.ssh/id_ed25519.pub
   ```

3. Copy the output and add it to GitHub under **Settings → SSH and GPG keys → New SSH key**. See [GitHub's SSH-key instructions](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account).
4. Test the connection:

   ```bash
   ssh -T git@github.com
   ```

If GitHub greets you by username, the setup is complete.

> Never share or copy `~/.ssh/id_ed25519`. Only `~/.ssh/id_ed25519.pub` is public.

## Using VS Code (Optional)

1. Install the [VS Code Remote - SSH extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh).
2. Run the `gcloud compute ssh` command above at least once. This creates and registers the local SSH key used by GCP.
3. Generate the exact underlying SSH command:

   ```bash
   [local] gcloud compute ssh INSTANCE_NAME --zone=ZONE --tunnel-through-iap --dry-run
   ```

4. Use the values from that output to add an entry to your local `~/.ssh/config`. A typical entry is:

   ```sshconfig
   Host osvm
       HostName INSTANCE_NAME
       User GCP_USERNAME
       IdentityFile ~/.ssh/google_compute_engine
       ProxyCommand gcloud compute start-iap-tunnel %h 22 --listen-on-stdin --project=PROJECT_ID --zone=ZONE
   ```

   Run `whoami` on the VM to find `GCP_USERNAME`.

5. Test the configuration:

   ```bash
   [local] ssh osvm
   ```

6. In VS Code, open the Command Palette and select **Remote-SSH: Connect to Host → osvm**.

## Troubleshooting

- Verify that the project, instance name, and zone are correct.
- If IAP reports a permissions error, contact course staff. Your account may need permission to use IAP and OS Login.
- If browser SSH works but the SSH-config entry does not, rerun the `--dry-run` command and use the exact username, key path, and proxy command it displays.
