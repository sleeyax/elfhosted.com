---
slug: Seafile
description: SABnzbd is a popular usenet client
upstream: https://www.seafile.com/en/support/
links:
works_with:
- LazyLibrarian
- Mylar
- Radarr
- Readarr
- Sonarr
links:
- name: subreddit
  uri: https://www.reddit.com/r/seafile/
---

# {{ page.meta.slug }}

Seafile is an open-source, cross-platform file-hosting software system. 

Files are stored on a central server and can be synchronized with personal computers and mobile devices through apps. 

Files on the Seafile server can also be accessed directly via the server's web interface.

Files on the server can be encrypted with client-side encryption, so that only authorized clients can view them.

{% include 'app.md' %}
{% include 'app_access.md' %}

## How do I use it?

Once you've enabled it in [your account](https://elfhosted.com/tenant/apps/0), launch the app via your dashboard.

Upon first login, you'll be prompted for credentials. Your credentials have been pre-setup as follows:

* User: `<username>@elfhosted.com`
* Password: `changeme`

You should immediately navigate to settings, and change your password. Ideally, let the `<username>@elfhosted.com` user remain unused as a backup admin user, and add your own independent admin user.

## Can I access synced data from my other apps?

**No**. Seafile uses git-like commits to track the sync state of files, which is why it's so fast, but consequently the files are unreadable in any other app. If you want to transfer files for the purposes of accessing with other apps (*i.e. streaming with [Jellyfin][jellyfin]*), then use [File Browser][filebrowser] or [Minio][minio].

## Can you see my files?

**No**. All we can see are blobs. And if you enable encryption on your libraries, all we'll see are encrypted blobs!

{% include 'app_footer.md' %}