<!--
  Copyright (c) 2020-2021 aetheryx & Cynthia K. Rey
  This work is licensed under a Creative Commons Attribution-NoDerivatives 4.0 International License.
  https://creativecommons.org/licenses/by-nd/4.0
-->

# Staying safe
Powercord has a lot of built-in security and self-defense features to ensure users safety. However, no solution
is perfect and malicious plugins may find their way through our protections and try to cause harm to you.

>danger
> In all cases, if you are suspecting your account to be at risk, change your password **immediately**, even if you
> have two factor authentication enabled. Also consider regenerating your 2FA backup codes, for additional safety.

## Common tips
### Install plugin from official sources
Plugins from official sources are more likely to be trustworthy and are actively reviewed by the staff to ensure no
malicious plugins enter the store. Malicious plugins could steal your account and cause widespread damage.

### Stay up to date
We constantly work on improving Powercord's internal security and the plugins do too. By keeping an up-to-date
Powercord installation, you ensure you're running with the most robust protection against thief.

### Be careful of social engineering
This is a rule valid universally, but you should especially be careful when it comes to modding. A malicious piece
of code can quickly go and steal your data, as mentioned in the first point.

When people tell you to modify some core files and you do not understand what you're doing, you should stop and not
do what they tell you! Attackers may want you to intentionally break our security layers and then attempt to steal
your account.

## How is Powercord secure?
Glad you asked! For the past years, most mods operated in a similar way, previous versions of Powercord included, where
plugins were simply thrown away as-is and then could do whatever they want. This is cool and simple, but raises a lot
of security concerns as plugins could access **anything**. Your Discord authentication token, personal information,
in some cases files, and the list goes on.

New versions of Powercord take a different approach. Powercord places itself as a supervisor and sits in between
plugins with Discord. By sitting in this position, it allows us to intercept and inspect everything plugins are
doing, allowing us to put strong restrictions on data they should never access and prevent them from doing
certain actions on your behalf.

This not only ensures integrity of your personal Discord data, but plugins are also protected from each others. If
you have a plugin using a secret API key stored in its settings, others plugins can't access this setting as Powercord
will not allow this to happen.

We hope this new approach increases the level of security of users and allow more peace of mind. This approach also
lets us watch what plugins do and inform developers of potential anti-patterns, slow functions, and various other
details that can give plugin developers deep insight on how to make their plugin smooth and pleasing to use. It's a win
for everyone.
