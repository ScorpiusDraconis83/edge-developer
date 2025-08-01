---
title: Tracking prevention in Microsoft Edge
description: The Microsoft Edge tracking prevention feature.
author: MSEdgeTeam
ms.author: msedgedevrel
ms.topic: conceptual
ms.service: microsoft-edge
ms.date: 01/07/2021
---
# Tracking prevention in Microsoft Edge

The tracking prevention feature in Microsoft Edge protects users from online tracking by restricting the ability of trackers to access browser-based storage as well as the network.

The tracking prevention feature is built to uphold the Microsoft Edge browser privacy promise, while also ensuring that there is no impact by default to website compatibility or the economic viability of the web.

See also:
* [Welcome to Microsoft Edge - A smarter way to protect](https://www.microsoft.com/edge/welcome)<!-- redir'd to this from [browser privacy promise](https://microsoftedgewelcome.microsoft.com/privacy) -->
* [Microsoft Edge Privacy Whitepaper](/legal/microsoft-edge/privacy)


<!-- ====================================================================== -->
## Levels of tracking prevention

Microsoft Edge currently offers users three levels of tracking prevention, which are selected by navigating to `edge://settings/privacy`.

![Three settings of tracking prevention](./tracking-prevention-images/tracking-prevention-settings.png)

1. **Basic** - The least restrictive level of tracking prevention that is designed for users who enjoy personalized advertisements and who don't mind being tracked on the web.  Basic only protects users against malicious trackers such as fingerprinters and cryptominers.

1. **Balanced (Default)** - The default level of tracking prevention that is designed for users who want to see less personalized advertisements while minimizing the risk of compatibility issues as they browse the web.  Balanced aims to block trackers from sites that users never engage with.

1. **Strict** - The most restrictive level of tracking prevention that is designed for users who are okay trading website compatibility for maximum privacy.

The tracking prevention feature in Microsoft Edge is made up of three main components that work together to determine whether a specific resource from a website should be classified as a tracker and blocked.  The components are as follows:

* **Classification** - The way Microsoft Edge determines whether a URL belongs to a tracker.

* **Enforcement** - The actions taken to protect Microsoft Edge users from URLs classified as trackers.

* **Mitigations** - The mechanisms provided to ensure user-specified favorite sites still work, while offering strong default protection.

Each of the components are explored and explained in detail on this page.


<!-- ====================================================================== -->
## Classification

The first component of the tracking prevention feature in Microsoft Edge is classification.  To classify online trackers and group them into categories, Microsoft Edge uses the [disconnect.me](https://disconnect.me) website's open source [Tracker Protection lists](https://github.com/disconnectme/disconnect-tracking-protection).  The Tracker Protection lists are delivered via the **Trust Protection Lists** component, which is listed in `edge://components`.  After being downloaded, the lists are stored on disk where you can use them to determine whether or how a particular URL is classified.

To determine if a URL is considered a tracker by the classification system in Microsoft Edge, a series of host names are checked, starting with an exact match and then proceeding to check for partial matches for up to four labels beyond the top-level domain.  If any of the host names in the series match with a host name on the disconnect.me website's Tracker Protection lists, Microsoft Edge proceeds with evaluating enforcement actions to prevent users from being tracked.


<!-- ------------------------------ -->
#### Example

For the URL `https://a.subdomain.of.a.known.tracker.test/some/path`, the tested host names are:

* `a.subdomain.of.a.known.tracker.test`
* `subdomain.of.a.known.tracker.test`
* `of.a.known.tracker.test`
* `a.known.tracker.test`
* `known.tracker.test`
* `tracker.test`


<!-- ====================================================================== -->
## Enforcement

To provide protection from tracking actions on the web, Microsoft Edge takes two enforcement actions against classified trackers:

*  **Restrict storage access** - If a known tracking resource tries to access any web storage where it may try to persist data about the user, Microsoft Edge blocks that access.  This includes restricting the ability for that tracker to get or set cookies as well as access storage APIs such as `IndexedDB` and `localStorage`.

*  **Block resource loads** - If a known tracking resource is being loaded on a website, Microsoft Edge may block that load before the request reaches the network depending on compatibility impact of the load and the tracking prevention setting a user has set.  Blocked loads may include tracking scripts, pixels, iframes, and more.  This prevents any data potentially being sent to the tracking domain and may even improve load times and page performance as a side effect.

A user may choose the page info flyout icon on the left side of the Address bar to find out which trackers were blocked on a specific page:

![Blocked trackers in the page info flyout](./tracking-prevention-images/page-info-flyout.png)

How the enforcements are applied depends on what level of tracking prevention the user selected and the mitigations that may apply.


<!-- ====================================================================== -->
## Mitigations

To ensure that web compatibility is preserved as much as possible, Microsoft Edge has three mitigations to help balance enforcements in specific situations:
* [The Org Relationship mitigation](#the-org-relationship-mitigation)
* [The Org Engagement mitigation](#the-org-engagement-mitigation)
* [The CompatExceptions list](#the-compatexceptions-list)


#### The concept of an _Organization_ (_Org_)

The [disconnect.me](https://disconnect.me) website maintains an [entities.json](https://github.com/disconnectme/disconnect-tracking-protection/blob/master/entities.json)<!-- changing master to main doesn't work 09/16/2024 --> list that defines groups of URLs that are owned by the same parent organization/company.  The tracking prevention feature in Microsoft Edge uses the `entities.json` list in both the Org Relationship mitigation and the Org Engagement mitigation, to minimize the occurrence of compatibility issues caused by tracking prevention affecting cross-organizational requests.


<!-- ------------------------------ -->
#### The Org Relationship mitigation

Several popular websites maintain both websites and Content Delivery Networks (CDNs) to serve static resources and content to those sites.  To ensure that these types of scenarios aren't affected by tracking prevention, Microsoft Edge exempts a site from tracking prevention when the site is making third-party requests to other sites owned by the same parent organization (as defined in the disconnect.me [entities.json list](https://github.com/disconnectme/disconnect-tracking-protection/blob/master/entities.json)).


<!-- ---------- -->
###### Example

An organization named Org1 owns the domains `org1.test` and `org1-cdn.test`, as defined in the disconnect.me `entities.json` list.  Imagine that `org1-cdn.test` is classified as a tracker and would normally have tracking prevention enforcements applied to it.  If a user visits `https://org1.test` and the site tries to load a resource from `https://org1-cdn.test`, Microsoft Edge doesn't take any enforcement actions against requests made to `org1-cdn.test` even though it isn't a first-party URL.  If another URL that isn't part of the Org1 organization tries to load that same resource, however, then the request would be subject to enforcements, because it isn't part of the same organization.

Even though this relaxes tracking prevention enforcements for sites that belong to the same organization, it is unlikely that this introduces a high amount of privacy risk, since such organizations are able to determine which sites/resources you have accessed on `https://org1.test` as well `https://org1-cdn.test` using internal back-end data.


<!-- ------------------------------ -->
#### The Org Engagement mitigation

The Org Engagement mitigation was created to minimize compatibility risks introduced by tracking prevention by ensuring that sites owned by organizations that users sufficiently engage with continue to work as expected across the web.  It makes use of the [Site Engagement](https://www.chromium.org/developers/design-documents/site-engagement) Service to relax enforcements whenever a user has established an ongoing relationship (currently defined by a site engagement score of 4.1 or greater) with a given site.

The Org Engagement mitigation is currently only applied in Balanced mode, so that Microsoft Edge is offering the highest possible protections for users who have opted into Strict.


<!-- ---------- -->
###### Example

An organization named Social Org owns the domains `social.example` and `social-videos.example`.

Users are considered to have a relationship with Social Org if they have established a site engagement score of 4.1 or greater with any one of domains owned by Social Org.

If another site, `https://content-embedder.example`, includes third-party content (say an embedded video from `social-videos.example`) from any of the domains owned by Social Org that would normally be restricted by tracking prevention enforcements, the site is exempt from tracking prevention enforcements as long as the user's site engagement score with domains owned by Social Org is maintained above the threshold.

If a site doesn't belong to an organization, a user must establish a site engagement score of 4.1 or greater with it directly before any storage access/resource load blocks imposed by tracking prevention are relaxed.


<!-- ------------------------------ -->
#### The CompatExceptions list

Based on recent user feedback Microsoft received, Microsoft Edge maintains a small list of sites (most of which are in the Disconnect Content category) that were breaking due to tracking prevention despite having the above two mitigations in place. Sites on this list are exempt from tracking prevention enforcements.  The list can be found on disk at the [locations](#determining-whetherhow-a-particular-url-is-classified) described below.  Users may override entries on it using the **Block** option in `edge://settings/content/cookies`.

To avoid maintaining this list moving forwards, Microsoft is currently working on the [Storage Access API](https://github.com/MicrosoftEdge/MSEdgeExplainers/blob/master/StorageAccessAPI/explainer.md) in the open-source codebase.  The Storage Access API gives site developers a way to request storage access from users directly, providing users with more transparency into how their privacy settings are affecting their browsing experience, and giving site developers controls to quickly and intuitively unblock themselves.

After the Storage Access API is implemented, Microsoft will deprecate the CompatExceptions list and reach out to the affected sites both to make them aware of the issues, and to request that they use the Storage Access API moving forward.


<!-- ====================================================================== -->
## Current tracking prevention behavior

The following are the enforcement actions and mitigations that are applied to each category of classified tracker in Microsoft Edge:

| Level | Advertizing | Analytics | Content | Cryptomining | Fingerprinting | Social | Other | Same Org Mitigation | Org Engagement Mitigation |
| - | - | - | - | - | - | - | - | - | - | - |
| **Basic** | - | - | - | B | B | - | - | Enabled | N/A |
| **Balanced** | S | - | S | B | B | S | S | Enabled | Enabled |
| **Strict** | B | B | S | B | B | B | B | Enabled | Disabled |

About the table:
*  The column headers are the categories of trackers as defined by the tracking protection list categories that are listed in disconnect.me [services.json](https://github.com/disconnectme/disconnect-tracking-protection/blob/master/services.json).
*  The left column lists the three levels of tracking prevention in Microsoft Edge (Basic, Balanced, and Strict).
*  `S` indicates that storage access is blocked.
*  `B` indicates that both storage access and resource loads (such as network requests) are blocked.
*  A hyphen (`-`) indicates that no block is applied to either storage access or resource loads.

The Org Engagement mitigation doesn't apply to the Cryptomining or Fingerprinting categories.

Strict mode blocks more resource loads than Balanced.  The blocking of more resource loads may result in Strict mode appearing to block less tracking requests than Balanced since the trackers making the requests are never loaded.

The Fingerprinting column in [Current tracking prevention behavior](#current-tracking-prevention-behavior) refers to trackers that are on the Fingerprinting list in addition to another list.  Trackers that appear on solely on the Fingerprinting list are considered non-malicious fingerprinters and aren't blocked.


<!-- ------------------------------ -->
#### InPrivate behavior

By default, InPrivate windows in Microsoft Edge use the same tracking prevention settings as regular windows.

To always apply the "Strict" level of tracking prevention to InPrivate windows:

1. Go to `edge://settings/privacy`.
1. Click **Always use "Strict" tracking prevention when browsing InPrivate**.


<!-- ====================================================================== -->
## Determining whether/how a particular URL is classified

To determine whether a specific URL is classified as a known tracker:

1. Open DevTools and open the **Console** tool.

1. Refresh the webpage.

   You may want to clear **Cookies and other site data** first to reset site engagement scores and ensure a completely clean slate.

1. Look for any messages that read `Tracking Prevention blocked access to storage for <URL>`.  You can expand the messages to see the individual URLs that were blocked.

1. If you need to determine which category a specific blocked site is in, the easiest way to do this is to search for it in the disconnect.me [services.json](https://github.com/disconnectme/disconnect-tracking-protection/blob/master/services.json) list.  The entries are alphabetized, so scrolling to the top of a block of site entries enables you to find the specific category for a particular site.


<!-- ------------------------------ -->
#### Access tracking prevention lists stored on disk

If you need to access the tracking prevention lists that are stored on disk, each may be found in one of two locations:

**Component-based updates** - The lists that are downloaded from the "Trust Protection Lists" component

Windows: `%LOCALAPPDATA%\Microsoft\Edge <OptionalChannelName>\User Data\Trust Protection Lists`

macOS: `~/Library/Application Support/Microsoft Edge <OptionalChannelName>/Trust Protection Lists`

**Installation directory** - The lists that are bundled with the Microsoft Edge Installer.  If you selected a different installation directory, your exact paths may be different.

Windows: `%PROGRAMFILES(x86)%\Microsoft\ Edge <OptionalChannelName>\Application<Version>\Trust Protection Lists`

macOS: `/Applications/Microsoft Edge.app/Contents/Frameworks/Microsoft Edge Framework.framework/Libraries/Trust Protection Lists`


<!-- ====================================================================== -->
## Blocking or allowing specific trackers for debugging purposes

Microsoft Edge only exposes an option to disable tracking prevention enforcements from running on a specified site.  This option is accessed via the page info flyout, or through the `edge://settings/privacy/trackingPreventionExceptions` page.

The **Block** and **Allow** options on the `edge://settings/content/cookies` page can be used to allow or deny specific domains access to storage such as cookies and other browser storage mechanisms.  This is useful for debugging site issues that are caused by tracking prevention enforcements blocking access to storage for a specific site.


<!-- ====================================================================== -->
## See also
<!-- all links in article -->

* [Welcome to Microsoft Edge - A smarter way to protect](https://www.microsoft.com/edge/welcome)
* [Microsoft Edge Privacy Whitepaper](/legal/microsoft-edge/privacy)
* [Storage Access API](https://github.com/MicrosoftEdge/MSEdgeExplainers/blob/master/StorageAccessAPI/explainer.md)

disconnect.me:
* [disconnect.me](https://disconnect.me) 
* [Tracker Protection lists](https://github.com/disconnectme/disconnect-tracking-protection)
* [entities.json](https://github.com/disconnectme/disconnect-tracking-protection/blob/master/entities.json) list - defines groups of URLs that are owned by the same parent organization/company.
* [services.json](https://github.com/disconnectme/disconnect-tracking-protection/blob/master/services.json) - tracking protection list categories.
