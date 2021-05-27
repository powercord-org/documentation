<!--
  Copyright (c) 2020-2021 aetheryx & Cynthia K. Rey
  This work is licensed under a Creative Commons Attribution-NoDerivatives 4.0 International License.
  https://creativecommons.org/licenses/by-nd/4.0
-->

# Host a backend
You want to make an amazing plugin, but this plugins requires a server and a backend? We got you covered. If you need
a backend server for your plugin, we are willing to host it for you and hook you up with an *.powercord.dev address,
free of charge, provided your backend meets some requirements.

## Requirements
To keep our server hamsters alive, you must comply with the following requirements in order to be eligible:

 - Code must meet basic quality standards

We don't ask you to build enterprise-grade software, but we expect basic protection against common attacks and the
code must not be over-engineered.

 - Resource usage must be coherent

We don't have unlimited CPU and memory. Unless you have a memory leak or you're attempting to ship a crypto miner,
you should be fine.

 - Be licensed under an [OSI approved license](https://opensource.org/licenses)

This is for transparency reasons, as well as to ensure we are allowed to run your code on our server. Note that we
expect the code to be easily available (for instance on GitHub or on GitLab) so people can poke at it easily.

 - The installation procedure must be straightforward

If we have to install half a billion tools to run your backend, it's a problem. If you require some additional backend
services, you can dockerize your app.

 - You must not ship a self-updater, or any other form of backdoor

Backends run isolated from sensitive data and could not in theory do harm, but 40 years of computing taught us that
is not something you can 100% ensure. It also helps ensuring transparency.

## Notes
Note that community backend will have **no access** whatsoever to Powercord's MongoDB database or other services used,
as we use authentication and access control for those as a basic safety measure.

If you need to use a database to store some data, we can give you a dedicated user for the database that will be only
accessible by you. Make sure to provide an easy way for us to specify a Mongo DSN.

## Post-approval
Once you've been approved, we will host your service and it will become available on `yourservice.powercord.dev`. To
prevent abuse, we have a rate limit in place in our nginx server.

If you want to push an update, you'll have to let a Powercord Staff member know so we can roll out the update on our
server.

If your backend ends up showing up signs of unforeseen vulnerabilities, or a memory leak we will take the appropriated
steps to mitigate the issue (or temporarily shut down your backend until a solution can be found). We'll notify you
about this.
