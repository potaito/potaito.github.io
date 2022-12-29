---
layout: post
title:  "I moved from KeePass to Bitwarden"
date:   2022-12-28 14:56:00 +0100
categories: security
---

I moved from the offline-based KeePass, which is deemed to be one of the safest password managers by many, to [Bitwarden](https://bitwarden.com/), an online password managers that stores the vault in the cloud. For the longest time my assumption was, that any online password manager is inherently less secure than one that stores the password only locally. So why did I go throug hthis change?

![cover image](/images/pexels-bess-hamiti-36487.jpg)

_Photograph by [Bess Hamiti](https://www.pexels.com/@bess-hamiti-83687/)_

I have been using a password manager for the last 10 years or so. This post is not about why password managers are a crucial security tool for _everyone_ that browses the internet. I think that in the year 2022 we are all in vast agreement on that and are already using password managers. If you are not, read the article ["Password Managers don't have to be perfect, they jsut have to be better than not having one"](https://www.troyhunt.com/password-managers-dont-have-to-be-perfect-they-just-have-to-be-better-than-not-having-one/) by Troy Hunt.

## My old password strategy
I have been using KeePass first, and then KeePass2 and KeepasXC. [KeePass](https://en.wikipedia.org/wiki/KeePass) is a whole familiy of password managers that is extendable via plugins, is available on many popular platforms. In my case Linux, Android and Windows are the relevant ones. Initially I would only create new login entries while using my main computer, and the copy of the password vault on my phone would be read-only in case I needed to log in to something on the go. A few things have changed over the years though.

My strategy was therefore to modify passwords only on one main device, and then copy the database to other devices from time to time.

## Shortcomings of a local password manager
### Mobile apps require logins
Nowdays a lot of mobile apps have their motivation to do their job bound to the existence of a user account. Before I didn't really bother too much with these app accounts and re-used old passwords that were not part of the password manager routine. Setting up new accounts was therefore quite simple. After a while though I wanted to have every one of my online accounts inside the password manager. Why? For security reasons and out of convenience, because a password manager also serves as a ledger showing where I have registered accounts. With hundreds of accounts that we amass over the decades it's sometimes easy to forget where we left our names and email addresses.

Managing the accounts for android apps with a local password manager in my case meant, that I'd have to generate and store the password on my main computer and then use it when creating an account on my phone. I also tried different approaches. But the bottom line is, that I could not find a satisfying solution where I could _create_ passwords on my phone and have it synced with the database on my main computer.

### More devices than ever
The number of devices that are in use by me is higher than ever before: A destkop computer, a personal notebook, a work-related notebook, dual-boot with Linux and Windows on some of these devices and a smartphone. This means I'm using around 6 different environemnts for work and personal things, that sometimes need to access the same password manager.

### Manual syncing and versioning is messy
As a first adaptation to my old strategy, I started modifying the Keepass database file on all devices where I needed to. I would manually track the changes by appending the version to the file name and then re-uploading to a cloud storage provider. This seems like a good solution in theory. The caveat however is, that whenever I wanted to modify the Keepass database, I first had to make sure I downloaded the latest version of my database from the central cloud storage. Otherwise I would end up modifying the database in two different places "simultaneously" and would later have to somehow reconcile the changes.


## Risk factors for my online accounts 
I'm paranoid, no doubt. And I try to maintain my privacy the best I can. At some point I have to be honest though and ask myself what the biggest risks for my online accounts and their passwords really are. Years ago my answer would have been a clear "well, some bad guy wants to steal it". Now the list is a bit longer:

- I accidentally forget to sync my offline password vault to other devices
- I accidentally delete or overwrite the most recent version of my vault
- I re-use old passwords in accounts of mobile apps because I'm too lazy to sync the database on my phone before and after every change
- Something happens to me and somebody in my family would need the account credentials, even just the name or email address while I'm in the hospital or something.
- Some bad guy wants to steal my password

## Conclusion
I realized that my laziness was starting to become a problem. The way I managed my passwords with an offline password manager was too cumbersome. So much in fact, that I started taking short cuts by re-using old passwords. Eventually I would come back and change them (yeah, right!), but maybe not always (actually, never).

After accepting these facts, I also reconsidered online password managers. In fact, I reconsidered so much, that I started using one. Now I have most passwords in one place, they are in perfect sync accross all devices, and a person of trust has access to a select few passwords in case something happens. Since we are both using Bitwarden, this was fairly easy to set up. We just created our own private organization with a few shared passwords in it.

In summary, I believe that my security is now better off when considering all risk factors, and not just the one extreme case where there is a targeted cyber attack against myself. And why did I choose Bitwarden? Because I believe in Open-Source, especially when it's Software that is related to security.
