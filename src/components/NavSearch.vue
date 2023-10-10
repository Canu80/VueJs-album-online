<template>
  <div class="content-bar">
    <div class="title">Añadir nuevo video</div>
    <div class="nav-bar">
      <input class="nav-input" v-model="videoLink" placeholder="Pega el enlace del video" />
      <button class="nav-button" @click="agregarVideo">Añadir</button>
    </div>
  </div>
</template>

<script>
import { toRefs, reactive } from 'vue';

export default {
 
  setup(_, { emit }) {
    const { videoLink } = toRefs(reactive({ videoLink: '' }));
    const videos = reactive([]);
    const YOUTUBE_API = process.env.VUE_APP_YOUTUBE_API;
    
    const agregarVideo = () => {
      const videoId = obtenerVideoId(videoLink.value);

      if (!videoExiste(videoId)) {
        obtenerDetallesVideo(videoId)
          .then((video) => {
            if (video.title && video.description && video.duration) {
              emit('nuevo-video', video);
              videoLink.value = '';
              enviarVideoAlServidor(video);
            } else {
              alert('El objeto "video" no contiene todas las propiedades necesarias.');
            }
          })
          .catch((error) => {
            alert('Error al obtener detalles del video:', error);
          });
      } else {
        alert('El video ya está en la lista.');
      }
    };

    const enviarVideoAlServidor = async (video) => {
      try {
        video.thumbnailUrl = video.thumbnail;
        const response = await fetch('http://18.218.183.17:3000/api/videos/', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify(video),
        });
        if (response.ok) {
          console.log('Video guardado en el servidor.');
        } else {
          console.error('Error al guardar el video en el servidor:', response.statusText);
        }
      } catch (error) {
        console.error('Error al enviar el video al servidor:', error);
      }
    };
 
    const obtenerDetallesVideo = async (videoId) => {
      const apiKey = YOUTUBE_API;
      const apiUrl = `https://www.googleapis.com/youtube/v3/videos?id=${videoId}&key=${apiKey}&part=snippet,contentDetails`;

      try {
        const response = await fetch(apiUrl);
        const data = await response.json();

        const videoDetails = data.items[0].snippet;
        const videoContentDetails = data.items[0].contentDetails;

        const duration = obtenerDuracion(videoContentDetails.duration);

        return {
          id: videoId,
          title: videoDetails.title,
          description: videoDetails.description,
          thumbnail: videoDetails.thumbnails.default.url,
          duration: duration,
        };
      } catch (error) {
        throw new Error('Error al obtener detalles del video desde la API de YouTube');
      }
    };

    const obtenerDuracion = (durationString) => {
      const matches = durationString.match(/PT(\d+H)?(\d+M)?(\d+S)?/);

      let hoursValue = 0;
      let minutesValue = 0;
      let secondsValue = 0;

      if (matches[1]) {
        hoursValue = parseInt(matches[1].replace('H', ''));
      }

      if (matches[2]) {
        minutesValue = parseInt(matches[2].replace('M', ''));
      }

      if (matches[3]) {
        secondsValue = parseInt(matches[3].replace('S', ''));
      }

      // Restar un segundo
      if (secondsValue > 0) {
        secondsValue -= 1;
      } else if (minutesValue > 0) {
        minutesValue -= 1;
        secondsValue = 59;
      } else if (hoursValue > 0) {
        hoursValue -= 1;
        minutesValue = 59;
        secondsValue = 59;
      }

      // Formatear la duración
      const formattedHours = hoursValue > 0 ? `${hoursValue}:` : '';
      const formattedMinutes = minutesValue > 0 ? `${String(minutesValue).padStart(2, '0')}:` : '00:';
      const formattedSeconds = secondsValue > 0 ? `${String(secondsValue).padStart(2, '0')}` : '00';

      const formattedDuration = `${formattedHours}${formattedMinutes}${formattedSeconds}`;

      return formattedDuration;
    };

    const videoExiste = (videoId) => {
      return videos.some((video) => video.id === videoId);
    };

    const obtenerVideoId = (link) => {
      let videoId = '';

      const matchYoutuBe = link.match(/youtu\.be\/([a-zA-Z0-9_-]+)/);
      if (matchYoutuBe) {
        videoId = matchYoutuBe[1];
      }

      const matchYouTube = link.match(/youtube\.com\/.*[?&]v=([a-zA-Z0-9_-]+)/);
      if (matchYouTube) {
        videoId = matchYouTube[1];
      }

      return videoId;
    };

    return {
      videoLink,
      agregarVideo,
    };
  },
};
</script>


<style>
.content-bar {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding-top: 70px;
  height: 17vh;
}

.title {
  display: flex;
  width: 60%;
  justify-content: left;
  align-items: left;
  font-family: 'Roboto', sans-serif;
  font-size: 1.75rem;
  font-weight: 700;
  padding: 18px;
}

.nav-bar {
  display: flex;
  width: 60%;
  height: 100%;
}

.nav-input {
  flex: 1;
  padding: 8px;
  border: none;
  border: 1px solid #d0d0d0;
  border-radius: 5px 0 0 5px;
  outline: none;
  color: #111;
  width: 80%;
  padding-left: 20px;
  font-family: 'Roboto', sans-serif;
  font-size: 1rem;
  font-weight: 500;
}

::placeholder {
  font-family: 'Roboto', sans-serif;
  font-size: 1rem;
  font-weight: 500;
}

.nav-button {
  padding: 7px 20px;
  background-color: #136AE4;
  color: #fff;
  border: 1px solid #136AE4;
  border-radius: 0 10px 10px 0;
  cursor: pointer;
  box-shadow: -1px 1px 2px rgba(0, 0, 0, 0.3);
  width: 20%;
  font-family: 'montserrat', sans-serif;
  font-size: 1rem;
  font-weight: 600;
}

</style>
