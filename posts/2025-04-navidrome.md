<!--
.. title: I hosted my own music streaming service
.. slug: music-streaming
.. date: 2025-04-05
.. tags: diy, music, open-source, self-hosting
.. category: 
.. link:
.. description:
.. type: text
-->

My Spotify subscription is years old, I think it might easily be my oldest online service. I've been generally happy with it, save for the regular, unexpected changes to user interface. If Spotify was a person, it would be a roommate who studies interior design and randomly re-arranges your furniture _("this way is better, you'll see!")_

However, things have been getting worse: I hear more and more often about Spotify offering [bad deals][spotify] to niche, emerging artists. Possibly for this reason, my favourite songs tend to disappear. When the [Iconoclasts soundtrack][iconoclasts-ost] vanished from Spotify, leaving a big hole in my favourite Indie OST playlist, I decided it's time to take matters into my own hands...

Turns out it's extremely simple to host your own music streaming service! Follow along to find out what setup I ended up with.

<!--more-->

## Tools

Here's the tools I've picked:

* Software: [**Navidrome**][nd]. Indexes music from a folder and offers it through an accessible web-UI. Allows setting up multiple user profiles; each user sees the same music but can create personal playlists.
* Hosting: [**Pikapods**][pika]. I don't run a homelab (yet) so a hosting service like this is a perfect match for me, I can set up the app in a few clicks and I don't have to worry at all about administration. I estimate I'll pay around $31/year for Navidrome. Pikapods shares some hosting profits with Navidrome developers.
* Mobile app: [**Youamp**][youamp]. I didn't do any research here; I know Navidrome speaks the [subsonic protocol][subsonic] and there's multiple apps that support it, I picked the first one on F-Droid that worked for me.
* Music: I upload music to my Navidrome using SFTP. I bought some amazing albums like [this one][a_rival] on Bandcamp. I got [some more][nectoulin] from Patreon. I also had some game soundtracks bought on Steam, I added these too. Later I'm also going to rip some CDs that I recently bought off [a friend][piepsi] who was tidying up his collection.

## Reflections

It feels very refreshing to buy DRM-free music again like it's the 90's -- especially when I still have all the convenience of using a streaming service. I'm hoping that this time the money I paid will support the artists more directly.

Now that I went through the above steps, I'm hoping I can get my friends to also participate: we can all use the same Navidrome instance and we can share our music with each other. Check your local law; many countries allow sharing media with family and friends.

I don't expect this solution will 100% replace Spotify for me, but it already brought me a lot of joy. It is however very likely that I'll spend more time looking at (and directly supporting!) emerging indie artists, now that the technical barrier is gone.


[subsonic]: https://www.subsonic.org/pages/api.jsp
[youamp]: https://f-droid.org/en/packages/ru.stersh.youamp/
[pika]: https://pikapods.com
[nd]: https://www.navidrome.org/
[iconoclasts-ost]: https://konjak.bandcamp.com/album/iconoclasts-soundtrack-birdsong
[a_rival]: https://rivalrivalrival.bandcamp.com/album/crypt-of-the-necrodancer-the-melody-mixes
[nectoulin]: https://www.patreon.com/posts/mp3-unreal-album-82841349
[piepsi]: https://bandcamp.com/piepsi91
[spotify]: https://www.musicbusinessworldwide.com/confirmed-next-year-tracks-on-spotify-1000-plays/