---
layout: post
title: appengine-ringo-io-0.1.0-SNAPSHOT
date: 2011-02-27 22:50:11.000000000 +09:00
type: post
published: true
status: publish
categories:
- Programming
- RingoJS
---
![](/images/2011/02/27/ec8aa4ed81aceba6b0ec83b7-2011-02-27-ec98a4ed9b84-10-36-25.png)

Google App Engine Java에서는 일부 I/O class들이 사용 제한되어있다.

그런데 이를 사용하려고 시도한 프로젝트가 있는데 대표적인게 
[GaeVFS(a.k.a. Google App Engine Virtual File System)](http://code.google.com/p/gaevfs/)) 이다.

전에 이 프로젝트와 [appengine-java-io](http://code.google.com/p/appengine-java-io/) 프로젝트를 사용해 파일을 불러와서 사용해 봤는데, 매우 만족스러웠다.

그런데... 완벽하게 잘 다루려면 좀 더 들여다 봐야할듯...([Vosao CMS](http://code.google.com/p/vosao) 소스 보면 좀 알 수 있으려나...)

암튼 요즘 ringojs로 이것저것 해보는데, Ride 라는 프로젝트를 발견했다. 근데 Appengine에서 IO때문에 돌리지 못한다는 글이 있길래...

[appengine-java-io](http://code.google.com/p/appengine-java-io/)를 ringojs에서 사용할 수 있도록 package를 만들어봤다.

이 과정에서 [appengine-java-io](http://code.google.com/p/appengine-java-io/) 코드를 ringojs에서 사용할 수 있도록 일부 수정해야했다.

만든 package는 [appengine-ringo-io](http://github.com/kyungw00k/appengine-ringo-io) 라고 이름을 짓고, 이를 사용해 [Ride 프로젝트를 Appengine 위에 올려봤다](http://ringo-ride.appspot.com).

일단 파일 목록 잘 나오고 불러오는거 잘되는데... modify가 안된다. 동작하고 있다는거에 만족하고, 추후 수정해봐야겠다.

어떻게 해결해야햐나... 음.. =_=;;
