---
layout: single
title:  "Logging Gulp Errors"
date:   2018-09-15 19:18:00 -0500
categories: javascript
comments: true
tags: javascript gulp
---

While building Aurelia applications, I have occassionaly made changes that cause the gulp build process to crash.  It is very easy to add console logging to your gulp steps so you can see exactly what the problem is.  

{% highlight javascript %}
gulp.task('build-html', function() { 
  return gulp.src(paths.html) 
    .pipe(changed(paths.output, {extension: '.html'})) 
    .pipe(htmlmin({collapseWhitespace: true})) 
    .pipe(gulp.dest(paths.output)); 
    .pipe(htmlmin({collapseWhitespace: true}).on('error', function(e){ 
      console.log(e); 
    })) 
   .pipe(gulp.dest(paths.output)); 
}); 
{% endhighlight %}