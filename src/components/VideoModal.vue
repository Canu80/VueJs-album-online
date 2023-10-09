<template>
  <div>
    <div v-if="video" class="modal">
      <div class="modal-content">
        <div class="video-column">
          <iframe
            :src="'https://www.youtube.com/embed/' + video.id"
            frameborder="0"
            allowfullscreen
            class="video-frame"
          ></iframe>
        </div>
        <div class="info-column">
          <p class="title">{{ video.title }}</p>
          <div class="description-container">
            <p class="description">{{ video.description }}</p>
          </div>
          <button @click="cerrarModal" class="modal-close-button">X</button>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  props: {
    video: Object,
  },
  methods: {
    cerrarModal() {
      this.$emit("cerrar-modal");
    },
  },
};
</script>

<style scoped>

.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.7);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal-content {
  display: flex;
  width: 70%;
  background-color: white;
  padding: 20px;
  border-radius: 5px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: space-between;
  align-items: center;
  position: relative;
}

.video-column {
  flex: 1;
  margin-right: 20px;
}

.info-column{
  flex: 1;
  margin-right: 20px;
}

.title {
  display: flex;
  justify-content: left;
  align-items: left;
  padding: 0;
  color: #3C3E41;
  font-family: 'Roboto', sans-serif;
  font-size: 1.2rem;
  font-weight: 700;
}
.description-container {
  max-height: 200px;
  overflow-y: auto;
}

.description{
  font-family: 'Roboto', sans-serif;
  font-size: 1rem;
  font-weight: 400;
}

.video-frame {
  width: 100%;
  height: 45vh;
}

.modal-close-button {
  position: absolute;
  top: 5px;
  right: 5px;
  cursor: pointer;
  color: rgb(0, 0, 0);
  background: none;

  border: none;
  font-family: sans-serif;
  font-size: 1rem;
  font-weight: 800;
}

.modal-close-button:hover {
  color: #136AE4;
}
</style>
