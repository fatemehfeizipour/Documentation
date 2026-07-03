# Documentation
This section contains my personal technical notes and structured learning summaries from my AWS journey.
------------------------
SSH is different from SSH into an EC2 instance

Since you've been working with AWS, it's easy to confuse the two:

SSH to GitHub

Purpose: Communicate with GitHub repositories.

SSH to an EC2 instance

Purpose: Log in to a Linux server.

Although both use the SSH protocol, they're used for different purposes:

GitHub SSH → authenticate Git operations.
EC2 SSH → open a remote terminal session on a server.
How to check whether your Git repository uses SSH or HTTPS

Run:

git remote -v

If you see:

origin  git@github.com:your-username/repo.git (fetch)

origin  git@github.com:your-username/repo.git (push)

you're using SSH.

If you see:

origin  https://github.com/your-username/repo.git

you're using HTTPS.
----------------------------------------------
