---
layout: post
title: Manage Globus groups with the Python SDK
categories: [python, globus]
---

[Globus](https://globus.org) is the best tool to transfer quickly Terabytes data between Supercomputers because it automatically parallelizes the transfer to saturate the network (yeah I know I always simplify too much).

If you want to share a folder with collaborators, you can create a Globus endpoint and give them access, see [how to do that at NERSC](https://docs.nersc.gov/services/globus/#sharing-data-with-globus).

Now, it's handy to create a group and share the endpoint directly with the group instead of individual users.

The web interface of Globus allows you to create groups, but you have to add people one at a time, and this is where the Globus Python SDK comes handy.

Install `globus_sdk` with `pip` and [follow the tutorial to have it configured](https://globus-sdk-python.readthedocs.io/en/stable/tutorial.html).

Clone [my repository of scripts](https://github.com/zonca/globus-sdk-scripts), assuming you use [`gh`](https://cli.github.com/):

    gh repo clone zonca/globus-sdk-scripts

Create a file `globus_config.toml` with your client ID and the name of the group you already created with the web interface:

    CLIENT_ID = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
    GROUP_NAME = "My group name"

## Create the tokens

Run the authentication script:

    python get_access_tokens.py

This will ask to open a link and paste back a string.

This will save the authentication tokens in 3 TOML files, those are sensitive, DO NOT COMMIT to public repositories.

## Batch add members to group

I assume you have a `users.csv` file with a column named "Email Address".

    python add_users_to_group.csv

is going to read that CSV, then grab 50 emails at a time, contact the Globus API to get the members ID if they have one, otherwise they are just skipped.
Then it batch-adds them to the group.

## Contribute

Please leave feedback to the <https://github.com/zonca/globus-sdk-scripts> repository via issues or contribute improvements via Pull Requests.
