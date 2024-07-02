---
title: About Me
permalink: /about/
---

<img src="/images/youth.jpg" style="width: 400px; margin-bottom: 0px;">
<span style="font-size: 0.7em; margin-top: 0px;">*Hacking into the mainframe circa 2004*</span>

Hey there! I'm Nafeu. I'm a programmer, musician, digital artist, content creator and general technology enthusiast. I started in software in 2014 and haven't looked back since. So far in my career I've done the following:
- architected, built and deployed full-stack web applications from start to finish with modern api integrations (payments processing via Stripe, error monitoring, email alerts, etc.)
- worked on platforms with complex data visualizations (analytics dashboards, internal system monitoring dashboards, various interactive charts, realtime data display)
- done freelance software consulting with clients in academic research, restaurant/catering management, speech therapy, and more

Software is my most passionate hobby and I've got A LOT of side projects. I also love reading, creating generative art in Processing, producing electronic music and playing video games, here are some quick facts:
  - I like to play competitive first-person shooters like Counter-Strike and Overwatch. I practically grew up on Doom and Unreal Tournament 99 (feel free to add me on Steam: [PhraktureMusic](https://steamcommunity.com/id/f1r3freak))
  - I've been making progressive breaks, liquid drum'n'bass and IDM using Cubase and Ableton under the artist name [Phrakture](http://music.phrakture.com) since 2006, indie electronic music under the artist name [Nafeu](https://open.spotify.com/artist/5NhwrCkzOykT6SdxGzwEtL?si=zgwxIaxsS2iIFZZ8GYvuuQ)
  - I recently completed the original videogame soundtrack for [Lorn's Lure](https://x.com/_rubeki)

Follow me on [Goodreads](http://www.goodreads.com/nafeu) to see what I've been digging in to lately and feel free to recommend a good book while you are at it!

## <a name="dev"></a>Development

I specialize in sleek, high-interactivity modern user interfaces for ETL dashboards, forms, configuration and data visualizations. However I've also spent quite some time building complex multi-step API integrations. Check out my work experience below:

#### <a name="work"></a>Work Experience

**Senior Front-End Engineer** - Tenjin Inc., _Remote - January 2021 to Present_
- Implement and maintain various complex front-end features for Tenjin.io customer dashboard in a Rails/Postgres/ReactJS tech stack
- Write and maintain comprehensive developer documentation, manage front-end candidate technical assessments
- Host many internal events ranging from social cafés, lightning talks, open-source contribution days, user personas creation, etc.
- Contribute to Tenjin Engineering Blog on medium

**Full Stack Developer** - Pelmorex Corp., _Toronto, Ontario – January 2019 to December 2020_
- Used React (incl. ES6, Functional Components, Await/Async, Context, and more), Redux, Lodash, Material UI, MongoDB, Elasticsearch, Redis and various other modern full-stack technologies to perform bug fixes, optimizations, bug tracking and comprehensive feature implementations on EngageFront (DSP) for Console Team
- Participated in Scrum/Agile development life-cycle with related scrum ceremonies (estimation, sprint planning, retrospectives, etc.) and manage project work with JIRA. Collaborate with Data Science, Bidder and Infrastructure teams
- Built and maintained internal development scripts, easy to use CLIs for various development task automation (incl. application deploys), documentation and complex realtime monitoring tools
- Developed a deeper understanding of AdTech industry (general domain, reporting, DSP/DMPs, exchanges, publishers, advertisers, performance metrics, ad operations, etc.)

**Software Developer** – DIVE Networks, _Toronto, Ontario – Sep 2016 to Sep 2017_
- Used Botkit, Slack API (Web, RTM and Events), Python and Node.js to build a secure chat bot server and Slack integration that interacts with DIVE's content management system.
- Performed general bug fixing/debugging and feature work for Team DIVE, DIVE Player and DIVE Dashboard which utilize technologies, languages and frameworks such as Clojure, ClojureScript, Leiningen, Garden, Django (Python), React (Reagent, Reframe), Sentry and JavaScript.

**Software Developer (RA / Part-Time)** – University of Toronto, Centre for French & Linguistics, _Scarborough, Ontario – Aug 2016 to May 2018_
- Used D3.js, MEAN stack, Heroku and various open-source tools to build and maintain multiple educational and
teaching tools for use by students and faculty in the Linguistics Department. This includes a dynamic syntax tree
builder, a lecture presentation app with real-time audience interaction/participation features, an interactive IPA feature
explorer and more.

**Bluemix Specialist** – Ecosystem Development – IBM Canada, _Markham, Ontario – May 2016 to August 2016_
- Prototyped and built demos using IBM Bluemix (PaaS) and Watson APIs for use in presentations, workshops and general tech demonstrations.
- Ran workshops on IBM Bluemix internally and externally at hackathons, tech meetups, code schools, university startup incubators and client meetings.

**Full Stack Web and Application Developer** – Three Point Turn Inc. _Toronto, Ontario – Jun 2014 to Jan 2016_
- Performed extensive front-end and back-end development work on ASP.NET MVC5 web applications applying responsive front-end design practices, Service Oriented Architecture, using REST APIs, participating in client demos and collaborating with senior developers

#### <a name="projects"></a>Projects

<!-- style="background-image: url('{{ project.image }}');" -->

<div class="projects-list">
{% for project in site.projects %}
<div class="project">
  <div class="project-details">
    <div class="project-name">
      {{ project.name }}
      {% if project.source %}
      <a href="{{ project.source }}">[src]</a>
      {% endif %}
      {% if project.preview %}
      <a href="{{ project.preview }}">[preview]</a>
      {% endif %}
      {% if project.live %}
      <a href="{{ project.live }}">[live]</a>
      {% endif %}
      {% if project.details %}
      <a href="{{ project.details }}">[details]</a>
      {% endif %}
    </div>
    <div class="project-desc">{{ project.desc }}</div>
  </div>
  <div class="project-images">
    {% if project.images %}
      {% for src in project.images %}
        <image class="project-image" src="/images/{{ src }}" />
      {% endfor %}
    {% endif %}
  </div>
</div>
{% endfor %}
</div>

## <a name="volunteering"></a>Volunteering and Community Involvement

**Tech Enthusiast, Organizer** - [Introspective Code](http://github.com/introspective-code), _Toronto, Ontario – August 2014 to Present_

<div class="volunteering-info">
  <div class="volunteering-photo" style="background-image: url('/images/about-introspective-code-1.jpg');"></div>
  <div class="volunteering-photo" style="background-image: url('/images/about-introspective-code-2.jpg');"></div>
  <div class="volunteering-photo" style="background-image: url('/images/about-introspective-code-3.jpg');"></div>
</div>

**Introspective Code** is a code mentorship, hobbyist and networking community which was brought together by myself and one of my greatest mentors (who happens to also be my older brother) [Shahriyar Nasir](https://snasir.ca/about/). The community emerged in early 2014 when I noticed a few close friends of mine were seeking a common form of hands-on guidance in full-stack web and application development. The cohort that formed was a mix of 2nd year Computer Science, Statistics and Computer Engineering students from University of Toronto who at the time were seeking (or had just entered) their first serious software internship.

Since Shah had already graduated from University of Toronto's Computer Science program and had a few years of software engineering experience under his belt, he offered his time and expertise to help us get our feet off the ground. For the next few months he organized/led workshops for us which he hosted in his office building's meeting space at [Nulogy](https://nulogy.com/).

Since then, students and friends from our cohort now work in senior software development positions at companies like Microsoft, Google, Scotiabank, RBC, 1Password, and more. Some of which have gone on to build their own YCombinator startups. Our community now mainly exists through our Slack channel where new members are joining every few months.

[Check out our github organization page here.](https://github.com/introspective-code/)


**Nourishment Volunteer** – Recreation Therapy – West Park Healthcare Centre, _Toronto, Ontario – August 2013 to December 2014_

## <a name="education"></a>Education

Honours Bachelor of Science - Double Major in Computer Science and Linguistics, University of Toronto

## <a name="music"></a>Music

<img src="/images/music-banner.png">

I've been producing progressive breaks, progressive house, drum & bass, chill and IDM/Glitch since I was 9 years-old (2004) and have continued with my passion for music ever since. I've put out 70+ original works (including 5+ album/ep releases) under the artist name **Phrakture** and have gotten support from artists/DJs like Jaytech, Solarstone, Matt Darey, Andy Moor, and more [(here is a shoutout from Solarstone himself)](https://soundcloud.com/springtube/slang-and-technodreamer-hypnosis-phrakture-remix-support-by-solarstone).

I've also released indie-electronic, nusoul, chilltrap and chillwave music under the artist name **Nafeu** and have had my work featured on [Chill Masters](https://www.youtube.com/watch?v=AbzCs9eiy0Q).

I've done some contracted work for [Flixel](https://flixel.com/), producing music for their videos [Lindsey Adler Shoots a Wedding Cinemagraph](https://www.youtube.com/watch?v=pT1Jn86r-C4) and [The Flixel Startup Story](https://www.youtube.com/watch?v=__gRRRjhhaw). I've also contributed a custom percussion pack for [ORION: Step Sequencer](http://echocollectivefx.com/product/orion-drum-machine), created a ton of midi controllerism, beatboxing, performance, and production tutorial videos on my YouTube channel [youtube.com/phrakture](https://youtube.com/phrakture), and released a few free sample packs on [Freesound](https://freesound.org/people/Phr4kture/). I have also been featured on [OverClocked Remix](https://ocremix.org/) for my remix of ["Foregone Destruction"](https://ocremix.org/remix/OCR01976) from the Unreal Tournament Soundtrack.

My main DAW is Cubase but I heavily incorporate Ableton for sound design. I specialize in warm melodies, deep 5th & 7th chord atmospheres, glitch edits and layered basslines. I'm a frequent user of some of the classic Native Instruments VSTs like Pro-53, FM8, Reaktor, Battery, Massive, etc. I also love incorporating live guitar, vocals, and field recordings.

You can check out some of my work at the following links:

#### Phrakture

- [Phrakture.com](https://phrakture.com)
- [Bandcamp Discography](https://phrakture.bandcamp.com/music) - includes many self-released works
- [Beatport](https://www.beatport.com/artist/phrakture/99726)
- [Spotify](https://open.spotify.com/artist/4AlnXoFGT5zl3v85ScIOzK?si=ITGOTpZ7T1qTpVWC4NMvlQ)
- [Soundcloud](https://soundcloud.com/phrakture)
- [Freesound](https://freesound.org/people/Phr4kture/) - my free sample packs
- [Official Discography Info](https://www.discogs.com/artist/1364238-Phrakture?page=1) - maintained by Discogs

#### Nafeu

- [Spotify](https://open.spotify.com/artist/5NhwrCkzOykT6SdxGzwEtL?si=WpivnOBpRFWCvIp_ZR4Gig)
- [Soundcloud](https://soundcloud.com/nafeumusic)

## <a name="contact"></a>Contact

Personal or Software Related: [nafeu.nasir@gmail.com](mailto:nafeu.nasir@gmail.com)<br>
Music Licensing or Remix Inquiry: [phrakturemusic@proton.me](mailto:phrakturemusic@proton.me)



