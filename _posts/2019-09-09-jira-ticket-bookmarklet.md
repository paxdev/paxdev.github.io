---
  title: JIRA Ticket Bookmarklet
  header:
    subheading: Launch a ticket from the bookmark toolbar in your browser
  tags:
    - jira
    - productivity
    - devops   
---

Shave a couple of clicks off the time taken to open a JIRA ticket with this handy tip. Note that, while this post relates to using JIRA, the same process can be used for any hackable URL.

In Firefox right-click the bookmark toolbar and select "New Bookmark". For Chrome it's "Add Page"

![New bookmark](/assets/posts/jira-bookmarklets_new-bookmark.png)

Give the bookmark an appropriate name and then for the location (URL in Chrome) add the following link:

```html
data:text/html,<script>window.location=(`https://[[your-jira-url]]/browse/${prompt('Enter ticket number')}`)</script>
```

![Enter bookmark details](/assets/posts/jira-bookmarklets_bookmark-settings.png)

What we've done is tell the browser to open a prompt and then insert the output from that prompt into JIRA's hackable URL. The `data:text/html` link type is used instead of the traditional `javascript` in order for this to work in Chrome.

![Bookmark clicked](/assets/posts/jira-bookmarklets_when-clicked.png)

