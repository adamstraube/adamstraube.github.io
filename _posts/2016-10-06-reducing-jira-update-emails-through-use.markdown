---
layout: single
comments: true
title: Reducing JIRA Update Emails Through use of Custom Listeners and Scriptrunner
date: '2016-10-06T12:45:00.000+10:00'
author: Adam Straube
tags: JIRA
---

If you use Atlassian JIRA and have notifications enabled then you have likely thought "This is sending me too many Email's". After checking the configuration's for notifications you will notice that you are rather limited in what you can customise, for example what if you wanted to exclude a certain field from giving email updates?<br />
<br />Now JIRA is quite extensible, if you are willing to develop and compile some java, but not all of us are comfortable with doing this for a plethora of reasons. The next best thing is to use a plugin that allows for the creation of custom listeners. There are a handful on the Atlassian marketplace but I have chosen <a href="https://marketplace.atlassian.com/plugins/com.onresolve.jira.groovy.groovyrunner/server/overview">Scriptrunner</a>&nbsp;as it has been around for a long time and is backed by a sizable company (Adaptavist).<br /><br />
<i><br /></i><i>For this example we will stop notifications on any update to the 'Labels' field.</i><br /><br /><br />
First step is to create a new Event. Go to the JIRA Administration area and click on <b>System &gt; Events, </b>scroll down to the bottom of the page to <b>Add New Event. </b>Name the event something that you will remember and select the <b>Issue Updated</b>
&nbsp;Template. Add it.<br /><br />
{% include image.html img="/images/custom-event.png" url="/images/custom-event.png" caption="Adding the custom event in the administrative interface" %}
<br />
Next step is to add a Script Listener through Scriptrunner. Back in the Admin area in JIRA go to&nbsp;<b>Add-ons &gt;
</b>&nbsp;<b>Script Listeners, </b>under <b>Add New&nbsp;</b>click on <b>Fires an event when condition is true</b>.<br />
<br />Now you will have to decide what Project's that you want to perform this modification for, or select all of them. Put the following code into the Condition field:<br />
<br />

{% highlight ruby %}
changeItems.any {
  it.field != 'labels'
}
{% endhighlight %}

<span style="font-family: inherit;"><b><br />
</b></span>Finally you need to modify the Notification Scheme for the projects that you want this change to be performed on. In the Admin area again click on <b>
Issues &gt; Notification Schemes, </b>select the Notification scheme you want to change and scroll to <b>Issue Updated. </b>
Take note of who receives the notification, scroll down further to your new event, Click <b>Add </b>and put in the same users that you just noted.<br /><br />You're done!

