# Privacy Policy

Last updated: November 7, 2026

This policy, from Smartonlabs Inc. ("we"), covers the Libertas One apps for Android and iOS, the Libertas Hub,
Libertas Studio, and the Libertas network services that connect them.

**In short:** your data lives on your Hub. The apps talk to your Hub, end-to-end
encrypted. The apps contain no advertising or analytics. We do no user tracking, and we sell
or share no personal data.

## What stays on your Hub

Accounts, device settings, App data (such as a tax return), camera streams and
history are stored on the Hub you own or run. The Hub's administrator decides who
can see and change what. We cannot read it.

## Cloud accounts

A cloud account runs on a Hub we host for you instead of one in your home. During
the public beta, cloud account data is cleared regularly without prior notice.
Don't keep anything in a beta cloud account that you can't afford to lose.

To sign you in, our cloud bridge keeps the account identifier from your sign-in
provider (such as Google or GitHub), with the name and email address the provider
shares, and when you last signed in. We use them only to sign you in and to run
the service. Your App data stays on the hosted Hub.

You can delete a cloud account at any time with **Delete cloud account** in the
app's cloud sign-in screen, or on the bridge's `/account/delete` page. Deletion
removes the Hub user and all its data, and erases the name and email. We keep only
a minimal record of the deletion, so unfinished sign-up work can't bring the
account back.

## What the mobile apps use, and why

The apps ask for each permission only when a feature needs it. Nothing collected
through them is sent to us.

| Permission | Used for |
| --- | --- |
| Camera | Scanning Matter setup codes and App authorization QR codes; AI photo import when you choose it. |
| Microphone | Voice commands, and two-way audio with supported cameras. |
| Speech recognition | Turning a spoken command into text for your Hub. On iOS this uses Apple's speech recognition; on Android, the system speech service. |
| Bluetooth | Finding and pairing nearby Matter devices. |
| Local network and Wi-Fi | Discovering your Hub and devices, and connecting directly to camera streams. |
| Location | Identifying the current Wi-Fi network for a direct Hub connection, and setting your Hub's location and time zone. Location is sent only to your Hub. |
| Notifications | Alerts from your Hub and its Apps. |
| Face ID, fingerprint or biometrics | Confirming it's you before opening Hub administration. Biometric data never leaves your device's system. |

## Crash reports

The apps contain no crash-reporting code and send no crash data to us. When an app
crashes, the operating system handles it: Google collects crash reports from the
Android app through Google Play, and Apple collects them from the iOS app if you
share analytics with app developers. Both follow their own privacy policies and
your device's settings, and both show us these reports in their developer consoles.

The Hub sends no crash reports to us. Its crash records stay on the Hub, for the
Hub's own users.

## Connecting from outside your home

When your phone can't reach the Hub directly, our relay service passes messages
between them. Traffic stays end-to-end encrypted with keys we don't hold; the relay
delivers messages without being able to read them. To do this the service keeps
anonymous authentication keys for Hubs and app installs, and each Hub's current
network address. We log connections to the service to detect abuse and keep those
logs for up to 5 years. A Hub reachable at a fixed address doesn't need the relay.

## AI features

AI tool calls, AI Import and AI Search are optional. When the Hub's administrator
connects a language model, the Hub sends that model the content of the request you
make, such as the photo or PDF you import or the words you say, under that
provider's own privacy terms. Without a connected model, nothing is sent.

## Apps from other developers

Apps installed on a Hub can't reach outside services unless the Hub's administrator
grants network access to that App. Developers who exchange data with outside
services must say what they share and why.

## Updates and Plaza

The Hub checks for software updates, and Studio and the Hub query the Plaza App
catalog. These requests carry no personal data.

## Children

The apps aren't directed at children under 13, and we don't knowingly collect their
data.

## Deleting your data

See [Account and data deletion](support.md#account-and-data-deletion).

## Changes

We'll update the date above when this policy changes.

## Contact

Smartonlabs Inc. Questions about privacy: [support@smartonlabs.com](mailto:support@smartonlabs.com).
