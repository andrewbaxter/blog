# Identifiers and Descriptors

The terms identifier and descriptor have more general meanings than I'm using here, but I find them useful for contrasting two concepts that are frequently mixed up.

An **identifier** is a information used to uniquely refer to something.

A **descriptor** is a information used to give a human an intuitive sense of what something is.

## The problems

### 1. Descriptors are often used where identifiers are needed, and descriptors are bad identifiers.

- Coming up with unambiguous descriptors is hard.

  This blog - I couldn't figure out if it was a blog or some sort of collected essays thing, and I was hoping to have a more "this is what I worked on this week" type blog separately, later, which would more accurately be described "blog". I changed the repository name several times worrying about this.

  The repository name shows up in the URL, so if I chose wrong I'm left with two options: keep the bad name forever, or change it and change everything that referred to it, possibly getting things wrong and causing issues down the line if you missed one, and hoping external links get updated by people who care.

  So either the descriptor will be a bad descriptor (obscures what something is) or will not be stable and will therefore be a bad identifier.

- Even if a descriptor is unambiguous, it can become ambiguous later.

  If the west wing gets another access point, "west wing access point" is no longer unambiguous. And worse, you may not know which of the two access points was the one documents were originally referring to.

- The unambiguity of a descriptor can be relative.

  Suppose your company has a provisioning team, and some new work comes up where you need to tell someone in that team which server to use: "database server" may be ambiguous to you, but maybe multiple divisions have database servers and the provisioning team works with all of them - the descriptor doesn't function as an identifier in this communication.

Descriptors _are_ useful -- people need to know what things are, in the end. But descriptors should be easy to modify, or specify differently in different contexts and therefore cannot be the same as identifiers.

### 2. Does "name" mean "identifier" or "descriptor"?

The term "name" is ambiguous and is used to refer to either, or both. IMO it just shouldn't be used.

## Problems in the real world

This causes real problems, frequently.

- Human names

  Many systems, private and public, make assumptions about names: that your name doesn't change and that it has a fixed representation.

  Some countries require names in specific formats, not explicitly documented (for foreigners). For instance: first last, last first, first middle last, last first middle. They may have a form without a middle name field, and either not want a middle name, or want it added to the first name. I've had papers rejected for name format mismatches before.

  They may have a name length limit. You may have to truncate your name on physical papers to match the name length limit in the entry form previously used on a computer.

  Airport security matches passengers for "no-fly" lists based on name, which can end up with the wrong people being denied boarding.

  When marrying or divoricing, if you change your name, you need to update many related services or risk mismatches causing service denial. There may be things associated with your name that cannot be changed, like your name on published works. Changing your name may thus risk your professional reputation.

- Domain names

  People need a way to refer to servers other than IP addresses, because IP addresses can and do change frequently. IP addresses are tied to ISPs, so you can't take an IP with you if you move providers. Domain names were a level of indirection so you could have a persistent identifier with changing IPs.

  They were immediately munged with descriptors, causing all sorts of issues:

  - Despite the fact that they act like identifiers, browsers treat them like descriptors which makes them important for marketing
  - The marketing importance causes companies to replace domains when rebranding
  - Changing domain names breaks external links
  - Bad actors can take over the old domains and control traffic to broken links
  - Security, like OIDC-generated temporary credentials, may use the domain name as the key security factor. Changing domain names could allow unauthorized access.
  - Even if you don't change your domain name, you can lose the domain name due to registrar issues or legal IP actions, affecting technical operations. This is primarily driven to competition over domain names due to their marketing value.
  - Domain squatting, auctions, high prices: because of the marketing value most companies don't have a choice but to fight for domain names.

- URLs

  URLs change due to domain names as well as path routing changes, also causing broken links, broken caching and necessitating complex redirections.

  There's a common hack of including both a descriptor and an identifier in a URL, then having the website ignore the descriptor part. This works around some issues, but external software doesn't know about the equivalence. This breaks things like caching, where a browser sees a different URL and redownloads resources because of it.

  [Cool URIs don't change](https://www.w3.org/Provider/Style/URI)

- Email addresses

  Email addresses include domain names, so have all the issues with domain name changes. They're also typically prefixed with your own name, so have the issues with your name changing.

  Email addresses are used in professional contexts, where a real name confers some trust and a nonsense name has associations with fly-by-night operations, scammers, and hobby projects. In this case an email address is a descriptor.

  Email addresses are used as identifiers for logging in to many web services. This creates additional friction for changing email addresses.

  Because serving email services is difficult to do reliably (email reputation, availability, etc) most people rely on 3rd party email services: Google, Microsoft, their school, their employer, Fastmail, their mobile phone provider. All of these things can and will change too, forcing email address changes. For instance, Microsoft may change their email policies. You may switch jobs. A school may promise email addresses for life but then change policies later, or implement measures that make the email unreachable from where you now live.

- Misuse of "name"

  In GCP, instances (servers) have an "id". However, this id isn't used by any API calls, making it effectively useless as an identifier. It's not even clear how unique it is. The documentation says just

  > The unique identifier for the resource. This identifier is defined by the server.

  What server? The instance itself? Server isn't standard GCP terminology.

  The actual identifier, used by GCP's API, is the "name" field, which by contrast should be a descriptor.

## Exaples of successful separation

Generally speaking, separating identifiers and descriptors requires services to support it (protocols, client side software). It can't be done as a grass-roots movement.

But this has been accomplished, and when it's done it's great.

- AWS

  AWS doesn't do this across the board (I can only assume they have some monstrous technical debt), but in EC2 instances have an AWS-assigned random identifier like `i-abcdefgh1234` and a _separate_ descriptor using the `Name` tag.

  Anywhere unambiguity is needed, like API calls and audits, the ID is used.

  In the UI however, the `Name` tag is displayed by default many places

- Phone numbers, messaging services (What's App, Signal, Line, etc.)

  Phone numbers are long, meaningless identifiers that even people unfamiliar with technology are capable of using.

  On modern phones, the phone maintains an address book that replaces the identifier with a descriptor in most interactions (making calls, receiving calls). So users don't need to memorize phone numbers, they can just use the names they're familiar with in their social circles (even nicknames). The names can be updated at any time.

  All messaging services similarly have internal user IDs, but show descriptors in the UI. The descriptors are automatically managed by the UI.
