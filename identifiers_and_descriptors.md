# Identifiers and Descriptors

The terms identifier and descriptor have more general meanings than I'm using here, but I find them useful for contrasting two concepts that are frequently mixed up.

An **identifier** is information used to uniquely refer to something.

A **descriptor** is information used to give a human an intuitive sense of what something is.

## The problems

### 1. Descriptors are often used where identifiers are needed, and descriptors are bad identifiers.

- Good descriptors use information relevant to you, which might not be relevant to someone else.

  Suppose you administer a database server remotely. To you, this is "the database server". The database is hosted on a physical server in your company's server closet. To someone maintaining the hardware, this may be "the top server" (in the rack). When talking to people on your respective teams each of these are useful descriptors, but when talking cross-team they're meaningless.

  The relevance of the information to your circumstances makes the descriptor useful, but makes it not useful as an identifier.

- You won't come up with an unambiguous descriptor on your first try.

  This blog - I couldn't figure out if it was a blog or some sort of collected essays thing, and I was hoping to have a more "this is what I worked on this week" type blog separately, later, which would more accurately be described "blog". I changed the repository name several times worrying about this.

  The repository name shows up in the URL, so if I chose wrong I'm left with two options: keep the bad name forever, or change it and break everything that refers to it. Maybe I can fix some of them, but I just have to hope broken external references get fixed by someone else.

  In other words, either the descriptor will become a bad descriptor (obscures what something is) or will not be stable and will therefore be a bad identifier.

- Your unambiguous descriptor will become ambiguous someday.

  If the west wing gets another access point, "west wing access point" is no longer unambiguous. And worse, you may not know which of the two access points was the one documents were originally referring to.

Descriptors _are_ useful -- people need to know what things are, in the end. But descriptors should be easy to modify, or specify differently in different contexts and therefore cannot be the same as identifiers.

### 2. Does "name" mean "identifier" or "descriptor"?

Lots of technical specifications use the word "name". The term "name" is ambiguous and is used to refer to either, or both.

Frequently, the documentation isn't clear about how unique the "name" must be, or whether it's used in other contexts. IMO the word "name" just shouldn't be used (use "id" or "description", and make them independent pieces of data).

## Problems in the real world

So how does this actually cause problems?

- Human names

  Many systems, private and public, make assumptions about names: that your name doesn't change and that it has a fixed representation.

  Some countries require names in specific formats, not explicitly documented (for foreigners). For instance: first last, last first, first middle last, last first middle. There may be entry forms without a middle name field that wants you to omit it or append it to your first name. I've had papers rejected for name format mismatches before.

  They may have a name length limit. You may have to truncate your name on physical papers to match the name length limit in the entry form previously used on a computer.

  Airport security matches passengers for "no-fly" lists based on name, which can end up with the wrong people being denied boarding.

  When marrying or divoricing, if you change your name, you need to update many related services or risk mismatches causing service denial. There may be things associated with your name that cannot be changed, like your name on published works. Changing your name may thus risk your professional reputation.

  The above demonstrate issues caused by names being relative (different systems require different names), not being unique (no-fly) and not being stable (marriage, divorice). All of these could be solved by using ids instead of names (ex: government issued ids).

- Domain names

  People need a way to refer to servers (aside from by IP address), because IP addresses can and do change frequently. IP addresses are tied to ISPs, so you can't take an IP with you if you move providers. Domain names were a level of indirection so you could have a persistent identifier with changing IPs.

  But they were munged from the start with descriptors, causing all sorts of issues:

  - Despite the fact that they act like identifiers, browsers show them prominently (address bar, link hover) and treat them like descriptors which makes them important for marketing

  - The marketing importance causes companies to replace domains when rebranding

  - Changing domain names breaks external links

  - Bad actors can take over the old domains and control traffic to broken links

  - OIDC-generated temporary credentials use the domain name as the key security factor. Changing domain names could allow unauthorized access if a bad actor takes over an old domain used for OIDC.

  - Even if you don't change your domain name, you can lose the domain name due to registrar issues (hacks, social methods to steal domains) or legal IP actions. This is primarily driven due to competition over domain names due to their marketing value, and can be hard to detect because _because_ domains have descriptive worth, people frequently trade them for legitimate reasons.

  - Domain squatting, auctions, high prices: because of the marketing value most companies don't have a choice but to fight for domain names.

- URLs

  URLs change due to domain names as well as path routing changes, which also leads to broken links, broken caching and complex, failure prone redirections.

  There's a common hack of including both a descriptor and an identifier in a URL, then having the server ignore the descriptor part so if it receives a request for a page with a since-modified descriptor it can still return the correct content. This works around some issues, but external software might not know about the equivalence. This breaks things like caching, where a browser thinks the new URL refers to new content and redownloads resources because of it.

  See also [Cool URIs don't change](https://www.w3.org/Provider/Style/URI)

- Email addresses

  Email addresses include domain names, so have all the issues with domain name changes. They're also typically prefixed with your own name, so have the issues with your name changing.

  Email addresses are used in professional contexts, where a real name confers some trust and a nonsense name has associations with fly-by-night operations, scammers, and hobby projects. In this case an email address is a descriptor.

  Email addresses are used as identifiers for logging in to many web services, or simply for mail servers to deliver mail correctly. This creates additional friction for changing email addresses.

  Because serving email is difficult to do reliably (email reputation, availability, etc) most people rely on 3rd party email services like Google, Microsoft, their school, their employer, Fastmail, or their mobile phone provider. All of these things can and will change in the future, forcing you to change your email address. For instance, Microsoft may change their email pricing. You may switch jobs. A school may promise email addresses for life but then change policies later, or implement measures that make the email unaccessible from where you now live.

- Hashicorp Terraform/OpenTofu

  In Terraform, all resources get an "id" which also becomes how they're referred to in the Terraform code. The use in the code encourages people to treat them as descriptors so they can remember what they are. However once the resource is created, if you change the id, Terraform will think it's a new, unrelated resource (and that the previous resource is now gone) which will cause Terraform to delete and recreate it. This is disastrous for things like cloud data storage -- all your data will be destroyed.

  They have a hack of a workaround, a `moved` block which allows you to (if you remember) document the id changes so Terraform doesn't delete and recreate. Which you then need to remove before you next apply (be careful if you're working with multiple environments). And what if you need to swap names?

  A better solution comes in the form of their `CDK` which is a general purpose programming language wrapper that generates the Terraform code. Since resources can be assigned to variables, you're free to provide descriptors in the form of variable names and use something stable (like a random string) for the resource id!

  This isn't perfect of course. It's surprisingly hard to generate random ids in most editors (out of the box). Also, since the descriptors are only in the CDK code and not the generated Terraform code, when you run Terraform commands the output will only contain the random string ids which you'll then have to trace back manually.

- GCP's misuse of "name"

  In GCP, instances (servers) have an "id". However, this id isn't used by any API calls, making it effectively useless as an identifier. It's not even clear how unique it is. The documentation says just

  > The unique identifier for the resource. This identifier is defined by the server.

  What server? The instance itself? Server isn't standard GCP terminology.

  The actual identifier, used by GCP's API, is the "name" field, which by contrast anyone would expect to be a descriptor, since there _is_ an "id" field already...

## Exaples of successful separation

Generally speaking, separating identifiers and descriptors requires services to support it (protocols, client side software). It can't be done as a grass-roots movement.

But this has been accomplished, and when it is it's great.

- AWS

  AWS doesn't do this across the board (I can only assume they have some monstrous technical debt), but in EC2, instances have an AWS-assigned random identifier like `i-abcdefgh1234` and a _separate_ descriptor using the `Name` tag.

  Anywhere unambiguity is needed, like API calls and audits, the id is used. And in most places in the UI the `Name` tag is displayed alongside the id.

- Phone numbers, messaging services (What's App, Signal, Line, etc.)

  Phone numbers are long, meaningless identifiers that even people unfamiliar with technology are capable of using.

  On modern phones, the phone maintains an address book that replaces the identifier with a descriptor in most interactions (making calls, receiving calls). So users don't need to memorize phone numbers, they can just use the names they're familiar with in their social circles (even nicknames). The names can be updated at any time.

  All messaging services similarly have internal user ids, but show descriptors in the UI. The descriptors are automatically managed by the UI. In many you can even freely change the descriptor, like replacing "Parker" with "Brother" or assigning more familiar nick names.
