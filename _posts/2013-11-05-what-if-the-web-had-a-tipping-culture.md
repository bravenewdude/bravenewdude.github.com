---
layout: post
title: What if the Web had a "Tipping Culture"?
category: tutorial
tags: [opinion, web]
---
{% include JB/setup %}


Users don't like ads seeing ads on your site. You don't like putting ads on your site. Is there another way? I contend that we should try out something new: tipping.

What if a user finished using your incredible service or reading your great content, and found himself staring at something like this:


<link type="text/css" rel="stylesheet" href="//tipwidget.appspot.com/static/twtiny.css" />
<script src="//ajax.googleapis.com/ajax/libs/jquery/1.7.1/jquery.min.js"></script>
<script src="//www.google.com/jsapi"></script>
<script src="//tipwidget.appspot.com/static/tw.js"></script>
<div id="tipwidget">
  <div id="tipwidgettop">tip me</div>
  <div id="tipwidgetleft">If you like this service, consider showing your appreciation by leaving a tip. Thanks!</div>
  <div id="tipwidgetright">
        <div>Amount: $<input type="text" id="tipwidgetamount" placeholder="1.00" /></div>
        <div><input type="image" src="https://checkout.google.com/buttons/checkoutMobile.gif?w=118&h=24&style=trans&variant=no-text"
               alt="Tip" onclick="tipwidgetsupport('110370840777223333593');" /></div>
  </div>
  <span class="tipwidgetclear"></span>
  <div id="tipwidgetbottom">
      <div id="tipwidgetlogo">
        <a href="http://tipwidget.appspot.com" target="_b"><span class="tipwidgetc1">tip</span><span class="tipwidgetc2">widget</span></a>
      </div>
  </div>
</div>
<br />

Imagine that your users could immediately give you a dollar (or more) with the click of a button. What fraction of them would do it? The answer will vary a lot by your service and your user base. If one percent of your visitors tipped you one dollar each, that would bring in orders of magnitude more money than ads typically do.

Is one percent a reasonable estimate? I'm not sure. Maybe it's a little on the optimistic side, but I think it's probably in the ballpark for high-quality sites. It might take some time for people to get used to the idea of web-tipping, but if it was convenient enough a small proportion of web users would give small tips for useful and enjoyable content. That's all it would take for this to be a viable alternative to ads. It might also be a viable alternative to paid upgrades.

The "tip widget" above is fully functional (hint!), and it takes approximately one minute to [make one for yourself](http://tipwidget.appspot.com), assuming you already have a Google Merchant account. Then customize your widget to look however you want. Let's give this a try!

<br />

P.S. To make your tipwidget look like the one in this article, replace "tw.css" with "twtiny.css" in the first line of your html snippet.


<br />

<div style="text-align: center">

<div id="fb-root"></div>
<script>(function(d, s, id) {
	var js, fjs = d.getElementsByTagName(s)[0];
	if (d.getElementById(id)) return;
	js = d.createElement(s); js.id = id;
	js.src = "//connect.facebook.net/en_US/all.js#xfbml=1";
	fjs.parentNode.insertBefore(js, fjs);
}(document, 'script', 'facebook-jssdk'));</script>
<div class="fb-like" data-href="http://blog.quantitations.com/tutorial/2013/11/05/what-if-the-web-had-a-tipping-culture/" data-colorscheme="light" data-layout="box_count" data-action="like" data-show-faces="false" data-send="false"></div>

<div class="g-plusone" data-size="tall"></div>
<script type="text/javascript">
(function() {
	var po = document.createElement('script'); po.type = 'text/javascript'; po.async = true;
	po.src = 'https://apis.google.com/js/plusone.js';
	var s = document.getElementsByTagName('script')[0]; s.parentNode.insertBefore(po, s);
})();
</script>

<a href="https://twitter.com/share" class="twitter-share-button" data-count="vertical" data-dnt="true">Tweet</a>
<script>!function(d,s,id){var js,fjs=d.getElementsByTagName(s)[0],p=/^http:/.test(d.location)?'http':'https';if(!d.getElementById(id)){js=d.createElement(s);js.id=id;js.src=p+'://platform.twitter.com/widgets.js';fjs.parentNode.insertBefore(js,fjs);}}(document, 'script', 'twitter-wjs');</script>

</div>

