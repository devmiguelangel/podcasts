## Next.js

### Introducción

Nexts.Js es básicamente un pequeño framework para renderizar nuestras vistas desde el servidor (SSR), este fue construido sobre React, Webpack y Babel. Next nos permite arrancar un proyecto de una manera sencilla, siendo de cero configuración y solo teniendo que agregar unos pequeños comandos para tener el proyecto listo y a la orden.

```bash
➜  yarn init

➜  yarn add react react-dom next --exact

➜  mkdir pages
```

- **package.json**

```json
{
  "scripts": {
    "dev": "next",
    "build": "next build",
    "start": "next start"
  }
}
```

### Styled JSX

- Es mas acorde a React
- Evitamos problemas al escalar
- Escribimos CSS3 como siempre
- Solo aplica al componente
- Tampoco aplica a componentes internos y externos

- pages/index.js

```jsx
import React, { Component } from 'react'

export default class Home extends Component {
  render() {
    return (
      <>
        <h1>Hello Platzi!</h1>
        <p>Welcome to the Next.js course</p>
        <img src="/static/platzi-logo.png" alt="Platzi"/>

        <style jsx>{`
          h1 {
            color: red;
          }
          :global(p) {
            color: green;
          }
          img {
            max-width: 30%;
            display: block;
            margin: 0 auto;
          }
        `}</style>
        
        <style jsx global>{`
          body {
            background-color: white;
          }
        `}</style>
      </>
    )
  }
}
```

### Server Side Rendering

![Client Side Rendering](docs/Screen&#32;Shot&#32;2019-12-02&#32;at&#32;21.01.17.png)

![Server Side Rendering](docs/Screen&#32;Shot&#32;2019-12-02&#32;at&#32;21.03.02.png)

#### getInitialProps

Carga el contenido principal

```bash
➜  yarn add isomorphic-unfetch --exact
```

- pages/index.js

```jsx
import React, { Component } from 'react';
import Link from 'next/link';
import fetch from 'isomorphic-unfetch';

export default class Home extends Component {
  static async getInitialProps() {
    const req = await fetch('https://api.audioboom.com/channels/recommended');
    const { body: channels } = await req.json();

    return { channels };
  }

  render() {
    const { channels } = this.props;

    return (
      <>
        <header>Podcasts</header>

        <div className="channels">
        {
          channels.map(channel =>
            <Link href={`/channel?id=${channel.id}`}>
              <a className="channel">
                <img src={channel.urls.logo_image.original} alt=""/>
                <h2>{channel.title}</h2>
              </a>
            </Link>
          )
        }
        </div>
        
        <style jsx>{`
          header {
            color: #fff;
            background-color: #8756ca;
            padding: 15px;
            text-align: center;
          }
          .channels {
            display: grid;
            grid-gap: 15px;
            padding: 15px;
            grid-template-columns: repeat(auto-fill, minmax(160px, 1fr));
          }
          .channel {
            display: block;
            border-radius: 3px;
            box-shadow: 0px 2px 6px rgba(0, 0, 0, 0.15);
            margin-bottom: 0.5rem;
          }
          .channel img {
            width: 100%;
          }
          h2 {
            padding: 5px;
          }
        `}</style>
        <style jsx global>{`
          body {
            margin: 0;
            background-color: white;
            font-family: system-ui
          }
        `}</style>
      </>
    )
  }
}
```

- pages/channel.js

```jsx
import React, { Component } from 'react';
import Link from 'next/link';

export default class Channel extends Component {
  static async getInitialProps({ query }) {
    const { id: idChannel } = query;

    const [reqChannel, reqSeries, reqAudios] = await Promise.all([
      fetch(`https://api.audioboom.com/channels/${idChannel}`),
      fetch(`https://api.audioboom.com/channels/${idChannel}/child_channels`),
      fetch(`https://api.audioboom.com/channels/${idChannel}/audio_clips`),
    ]);

    const { body: { channel } } = await reqChannel.json();
    const { body: { audio_clips: audioClips } } = await reqAudios.json();
    const { body: { channels: series } } = await reqSeries.json();

    return { channel, audioClips, series };
  }

  render() {
    const { channel, audioClips, series } = this.props;

    return (
      <>
        <header>Podcasts</header>

        <div className="banner" style={{ backgroundImage: `url(${channel.urls.banner_image.original})` }}></div>

        <h1>{channel.title}</h1>

        {
          series.length > 0 && (
            <div>
              <h2>Series</h2>
              <div className="channels">
                {
                  series.map(serie => (
                    <Link href={`/channel?id=${ serie.id }`} key={serie.id}>
                      <a className="channel">
                        <img src={ serie.urls.logo_image.original } alt=""/>
                        <h2>{ serie.title }</h2>
                      </a>
                    </Link>
                  ))
                }
              </div>
            </div>
          )
        }

        <h2>Últimos Podcasts</h2>
        {
          audioClips.map(clip => (
            <Link href={`/podcast?id=${clip.id}`} key={clip.id}>
              <a className='podcast'>
                <h3>{ clip.title }</h3>
                <div className='meta'>
                  { Math.ceil(clip.duration / 60) } minutes
                </div>
              </a>
            </Link>
          ))
        }

        <style jsx>{`
          header {
            color: #fff;
            background: #8756ca;
            padding: 15px;
            text-align: center;
          }

          .banner {
            width: 100%;
            padding-bottom: 25%;
            background-position: 50% 50%;
            background-size: cover;
            background-color: #aaa;
          }

          .channels {
            display: grid;
            grid-gap: 15px;
            padding: 15px;
            grid-template-columns: repeat(auto-fill, minmax(160px, 1fr));
          }
          a.channel {
            display: block;
            margin-bottom: 0.5em;
            color: #333;
            text-decoration: none;
          }
          .channel img {
            border-radius: 3px;
            box-shadow: 0px 2px 6px rgba(0,0,0,0.15);
            width: 100%;
          }
          h1 {
            font-weight: 600;
            padding: 15px;
          }
          h2 {
            padding: 5px;
            font-size: 0.9em;
            font-weight: 600;
            margin: 0;
            text-align: center;
          }

          .podcast {
            display: block;
            text-decoration: none;
            color: #333;
            padding: 15px;
            border-bottom: 1px solid rgba(0,0,0,0.2);
            cursor: pointer;
          }
          .podcast:hover {
            color: #000;
          }
          .podcast h3 {
            margin: 0;
          }
          .podcast .meta {
            color: #666;
            margin-top: 0.5em;
            font-size: 0.8em;
          }
        `}</style>

        <style jsx global>{`
          body {
            margin: 0;
            font-family: system-ui;
            background: white;
          }
        `}</style>
      </>
    )
  }
}
```

- pages/podcast.js

```jsx
import React, { Component } from 'react';
import Link from 'next/link';

export default class Podcast extends Component {
  static async getInitialProps({ query }) {
    const { id: idPodcast } = query;

    const [reqPodcast] = await Promise.all([
      fetch(`https://api.audioboom.com/audio_clips/${idPodcast}.mp3`),
    ]);

    const { body: { audio_clip: clip }} = await reqPodcast.json();

    return { clip };
  }
  render() {
    const { clip } = this.props;

    return (
      <>
        <header>Podcasts</header>

        <div className="modal">
          <div className="clip">
            <nav>
              <Link href={`/channel?id=${clip.channel.id}`}>
                <a className='close'>&lt; Volver</a>
              </Link>
            </nav>

            <picture>
              <div style={{ backgroundImage: `url(${clip.urls.image || clip.channel.urls.logo_image.original})` }} />
            </picture>

            <div className='player'>
              <h3>{ clip.title }</h3>
              <h6>{ clip.channel.title }</h6>
              <audio controls autoPlay={true}>
                <source src={clip.urls.high_mp3} type='audio/mpeg' />
              </audio>
            </div>
          </div>
        </div>

        <style jsx>{`
          nav {
            background: none;
          }
          nav a {
            display: inline-block;
            padding: 15px;
            color: white;
            cursor: pointer;
            text-decoration: none;
          }
          .clip {
            display: flex;
            height: 100%;
            flex-direction: column;
            background: #8756ca;
            color: white;
          }
          picture {
            display: flex;
            align-items: center;
            justify-content: center;
            flex: 1 1;
            flex-direction: column;
            width: auto;
            padding: 10%;
          }
          picture div {
            width: 100%;
            height: 100%;
            background-position: 50% 50%;
            background-size: contain;
            background-repeat: no-repeat;
          }
          .player {
            padding: 30px;
            background: rgba(0,0,0,0.3);
            text-align: center;
          }
          h3 {
            margin: 0;
          }
          h6 {
            margin: 0;
            margin-top: 1em;
          }
          audio {
            margin-top: 2em;
            width: 100%;
          }

          .modal {
            position: fixed;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            z-index: 99999;
          }
        `}</style>

        <style jsx global>{`
          body {
            margin: 0;
            font-family: system-ui;
            background: white;
          }
        `}</style>
      </>
    )
  }
}
```
