<template>
  <div>
    <!-- Componente NavSearch -->
    <NavSearch @nuevo-video="agregarVideo" :videos="videos" />
    <div>
      <!-- Organizador de videos -->
      <div class="video-container">
        <div v-for="video in videos" :key="video.id" class="video-item" @click="mostrarModal(video)">
          <!-- Miniatura del video -->
          <div class="video-thumbnail-container">
            <img :src="video.thumbnail" alt="Miniatura del video" class="video-thumbnail" />
            <!-- Botón para eliminar el video -->
            <button class="delete-button" @click="abrirConfirmacionEliminar(video.id)">X</button>
          </div>
          <!-- Duración del video -->
          <div class="video-duration">{{ video.duration }}</div>
        </div>
      </div>
    </div>

    <!-- Componente VideoModal -->
    <VideoModal :video="videoSeleccionado" @cerrar-modal="cerrarModal" />

    <!-- Confirmación para eliminar un video -->
    <div class="delete-modal" v-if="showDeleteModal">
      <div class="delete-modal-content">
        <p class="text">¿Seguro que quieres eliminar este video?</p>
        <div class="delete-modal-buttons">
          <span class="x-button" @click="cerrarModal">X</span>
          <button class="cancelar" @click="cancelarEliminacion">Cancelar</button>
          <button class="eliminar" @click="confirmarEliminacion">Eliminar</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import '@fortawesome/fontawesome-free/css/all.css';
import NavSearch from './NavSearch.vue';
import VideoModal from "./VideoModal.vue";
import { ref, reactive } from 'vue';

export default {
  components: {
    NavSearch,
    VideoModal,
  },
  setup() {
    const videos = reactive([]);
    const modalVisible = ref(false);
    const videoSeleccionado = ref(null);
    const showDeleteModal = ref(false);
    let videoIdToDelete = null;
    const videoExiste = (videoId) => {
    return videos.some((video) => video.id === videoId);
    };

    const agregarVideo = (video) => {
      if (!videoExiste(video.id)) {
        videos.push(video);
      } else {
        alert('El video ya está en la lista.');
      }
    };

    const mostrarModal = (video) => {
      if (!showDeleteModal.value) {
        modalVisible.value = true;
        videoSeleccionado.value = video;
      }
    };

    const cerrarModal = () => {
      modalVisible.value = false;
      videoSeleccionado.value = null;
      showDeleteModal.value = false;
    };

    const abrirConfirmacionEliminar = (videoId) => {
      videoIdToDelete = videoId;
      showDeleteModal.value = true;
    };

    const cancelarEliminacion = () => {
      showDeleteModal.value = false;
    };

    const confirmarEliminacion = () => {
      if (videoIdToDelete) {
        eliminarVideo(videoIdToDelete);
        videoIdToDelete = null;
        showDeleteModal.value = false;
      }
    };

    const eliminarVideo = async (videoId) => {
      try {
        await eliminarVideoEnBaseDeDatos(videoId);
        const videoIndex = videos.findIndex((video) => video.id === videoId);
        if (videoIndex !== -1) {
          videos.splice(videoIndex, 1);
        }
      } catch (error) {
        console.error('Error al eliminar el video en el servidor:', error);
      }
    };

    const eliminarVideoEnBaseDeDatos = async (videoId) => {
      try {
        const response = await fetch(`http://localhost:8081/api/videos/${videoId}`, {
          method: 'DELETE',
        });
        if (response.ok) {
          console.log('Video eliminado en el servidor.');
        } else {
          console.error('Error al eliminar el video en el servidor:', response.statusText);
        }
      } catch (error) {
        console.error('Error al enviar la solicitud de eliminación al servidor:', error);
      }
    };

    return {
      videos,
      modalVisible,
      videoSeleccionado,
      showDeleteModal,
      mostrarModal,
      cerrarModal,
      abrirConfirmacionEliminar,
      cancelarEliminacion,
      confirmarEliminacion,
      eliminarVideo,
      agregarVideo
    };
  },
};
</script>

<style>

/* Contenedor de videos */
.video-container {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  margin-top: 20px;
  max-width: 60%;
  margin: 5% 22%;
}

.video-item {
  position: relative;
  width: calc(33.33% - 20px);
  margin-bottom: 20px;
}

.video-thumbnail-container {
  position: relative;
}

.video-thumbnail {
  width: 100%;
  height: auto;
}

.delete-button {
    position: absolute;
    top: 5px;
    right: 5px;
    cursor: pointer;
    color: rgb(255, 255, 255);
    background-color: rgb(0, 0, 0);
    border: none;
    font-family: sans-serif;
    font-size: 1rem;
    font-weight: 800;
}

.video-duration {
  position: absolute;
  bottom: 9px;
  right: 3px;
  background-color: rgba(0, 0, 0);
  color: white;
  padding: 1px 1px;
  border-radius: 2px;
}

/* Estilos para el modal de confirmación */
.delete-modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.delete-modal-content {
  display: flex;
  flex-direction: column;
  width: 50%;
  background: #f0f0f0;
  padding: 20px;
  border-radius: 15px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.3);
  text-align: center;
  position: relative;
}

.text{
  display: flex;
  width: 60%;
  justify-content: left;
  align-items: left;
  font-family: 'Roboto', sans-serif;
  font-size: 1.5rem;
  font-weight: 500;
  padding: 18px;
  color: #3C3E41;
}

.delete-modal-buttons {
  display: flex;
  justify-content: right;
  margin-top: 20px;
}

.x-button{
    position: absolute;
    top: 25px;
    right: 35px;
    cursor: pointer;
    color: #020202;
    background-color: none;
    border: none;
    font-family: 'montserrat', sans-serif;
    font-size: 1.2rem;
    font-weight: 800;
}

.delete-modal-buttons button {
  width: 30%;
  height: 8vh;
  margin: 0 10px;
  padding: 5px 10px;
  border: none;
  border-radius: 10px;
  cursor: pointer;
}

.delete-modal-buttons button.cancelar {
  background: transparent;
  font-family: 'montserrat', sans-serif;
  font-size: 1rem;
  font-weight: 700;
  color: #136AE4;
  border: 2px solid #136AE4;
}

.delete-modal-buttons button.eliminar {
  background: #136AE4;
  font-family: 'montserrat', sans-serif;
  font-size: 1rem;
  font-weight: 700;
  color: #ffffff;
}

</style>