<template>
  <div class="all">
    <div class="form-container">
      <h2>Formulario de Cifrado Asimétrico</h2>
      <form @submit.prevent="submitFormAsym">
        <div class="form-group">
          <label for="name">Nombre:</label>
          <input v-model="formData.name" placeholder="Ingresa tu nombre" required />
        </div>

        <div class="form-group">
          <label for="email">Email:</label>
          <input type="email" id="email" v-model="formData.email" placeholder="Ingresa tu email" required />
        </div>

        <div class="form-group">
          <label for="country">País:</label>
          <select id="country" v-model="formData.country" required>
            <option disabled value="">Selecciona un país</option>
            <option value="MX">México</option>
            <option value="US">Estados Unidos</option>
            <option value="ES">España</option>
          </select>
        </div>

        <button type="submit">Cifrar y Descifrar Asimétrico</button>
      </form>

      <div v-if="encryptedDataAsym">
        <h3>Datos Cifrados Asimétricos:</h3>
        <Textarea auto v-model="encryptedDataAsym" class="data-display" rows="5" autoResize></Textarea>
      </div>

      <div v-if="decryptedDataAsym">
        <h3>Datos Descifrados Asimétricos:</h3>
        <Textarea v-model="decryptedDataAsym" class="data-display" rows="5" autoResize></Textarea>
      </div>
    </div>

    <div class="form-container">
      <h2>Formulario de Cifrado Simétrico</h2>
      <form @submit.prevent="submitFormSym">
        <div class="form-group">
          <label for="nameSym">Nombre:</label>
          <input v-model="formDataSym.name" placeholder="Ingresa tu nombre" required />
        </div>

        <div class="form-group">
          <label for="emailSym">Email:</label>
          <input type="email" id="emailSym" v-model="formDataSym.email" placeholder="Ingresa tu email" required />
        </div>

        <div class="form-group">
          <label for="countrySym">País:</label>
          <select id="countrySym" v-model="formDataSym.country" required>
            <option disabled value="">Selecciona un país</option>
            <option value="MX">México</option>
            <option value="US">Estados Unidos</option>
            <option value="ES">España</option>
          </select>
        </div>

        <button type="submit">Cifrar y Descifrar Simétrico</button>
      </form>

      <div v-if="encryptedDataSym">
        <h3>Datos Cifrados Simétricos:</h3>
        <Textarea v-model="encryptedDataSym" class="data-display" rows="5" autoResize></Textarea>
      </div>

      <div v-if="decryptedDataSym">
        <h3>Datos Descifrados Simétricos:</h3>
        <Textarea v-model="decryptedDataSym" class="data-display" rows="5" autoResize></Textarea>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import forge from 'node-forge';
import Textarea from 'primevue/textarea';
import { defineComponent } from 'vue';

export default defineComponent({
  components: {
    Textarea
  },
  data() {
    return {
      formData: {
        name: '',
        email: '',
        country: '',
      },
      formDataSym: {
        name: '',
        email: '',
        country: '',
      },
      publicKey: '',
      privateKey: '',
      secretKey: '',
      encryptedDataAsym: '',
      decryptedDataAsym: '',
      encryptedDataSym: '',
      decryptedDataSym: '',
    };
  },
  methods: {
    generateKeys() {
      const { privateKey, publicKey } = forge.pki.rsa.generateKeyPair(2048);
      this.publicKey = forge.pki.publicKeyToPem(publicKey);
      this.privateKey = forge.pki.privateKeyToPem(privateKey);
      this.secretKey = forge.random.getBytesSync(16);
    },
    encryptDataAsym(data: string): string {
      const publicKey = forge.pki.publicKeyFromPem(this.publicKey);
      const encrypted = publicKey.encrypt(forge.util.encodeUtf8(data), 'RSA-OAEP');
      return forge.util.encode64(encrypted);
    },
    decryptDataAsym(encryptedData: string): string {
      const privateKey = forge.pki.privateKeyFromPem(this.privateKey);
      const decoded = forge.util.decode64(encryptedData);
      const decrypted = privateKey.decrypt(decoded, 'RSA-OAEP');
      return forge.util.decodeUtf8(decrypted);
    },
    encryptDataSym(data: string): string {
      const cipher = forge.cipher.createCipher('AES-CBC', this.secretKey);
      const iv = forge.random.getBytesSync(16);
      cipher.start({ iv });
      cipher.update(forge.util.createBuffer(data, 'utf8'));
      cipher.finish();
      const encrypted = cipher.output;
      return forge.util.encode64(iv + encrypted.getBytes());  // Concatenación correcta
    },
    decryptDataSym(encryptedData: string): string {
      const decoded = forge.util.decode64(encryptedData);
      const iv = decoded.slice(0, 16);  // IV es el primer bloque
      const encrypted = decoded.slice(16);  // Datos cifrados
      const decipher = forge.cipher.createDecipher('AES-CBC', this.secretKey);
      decipher.start({ iv });
      decipher.update(forge.util.createBuffer(encrypted));
      const isFinished = decipher.finish();
      return isFinished ? forge.util.decodeUtf8(decipher.output.getBytes()) : '';
    },
    submitFormAsym() {
      try {
        const formDataString = JSON.stringify(this.formData);
        this.encryptedDataAsym = this.encryptDataAsym(formDataString);
        this.decryptedDataAsym = this.decryptDataAsym(this.encryptedDataAsym);
        alert("Proceso de cifrado y descifrado asimétrico completo!");
      } catch (error) {
        console.error("Error en el cifrado asimétrico:", error);
        alert("Error en el cifrado asimétrico, revisa la consola.");
      }
    },
    submitFormSym() {
      try {
        const formDataStringSym = JSON.stringify(this.formDataSym);
        this.encryptedDataSym = this.encryptDataSym(formDataStringSym);
        this.decryptedDataSym = this.decryptDataSym(this.encryptedDataSym);
        alert("Proceso de cifrado y descifrado simétrico completo!");
      } catch (error) {
        console.error("Error en el cifrado simétrico:", error);
        alert("Error en el cifrado simétrico, revisa la consola.");
      }
    },
  },
  mounted() {
    this.generateKeys();
  },
});
</script>

<style scoped>
.form-container {
  margin-top: 10px;
  max-width: 500px;
  padding: 20px;
  border-radius: 8px;

  display: flex;
  flex-direction: column;
}

h2 {
  text-align: center;
  color: #333;
}


.form-group {
  margin-bottom: 15px;
}

.all {
  display: flex;
  flex-direction: row;
  gap: 40px;
}
@media (max-width: 700px) {
  .all {
    display: flex;
    flex-direction: column;
    
    height: 100vh;
    padding: 40px; /* Añade espacio para evitar que el contenido se pegue a los bordes */
    box-sizing: border-box;
  }

  .form-container {
    box-shadow: 0px 4px 5px 4px rgb(239, 239, 239);
    align-items: center;
    display: flex;
    flex-direction: column;
    width: 100%; /* Asegura que el formulario ocupe todo el ancho disponible */
    padding: 15px; /* Añade espacio interno para el contenido */
    margin-bottom: 20px; /* Espacio entre formularios */
    box-sizing: border-box; /* Evita que el padding altere el tamaño total */
  }

  .data-display {
    white-space: pre-wrap; /* Mantiene los saltos de línea */
    word-wrap: break-word; /* Rompe las palabras largas para que se ajusten al contenedor */
    background-color: #ffffff;
    padding: 10px;
    border-radius: 5px;
    /* overflow-x: auto; Permite desplazamiento horizontal si es necesario */
    box-sizing: border-box; /* Evita que el padding altere el tamaño total */
  }
}



label {
  display: block;
  margin-bottom: 5px;
  color: #555;
}

.data-display {
  white-space: pre-wrap;
  /* Permite que las líneas largas se rompan */
  word-wrap: break-word;
  /* Asegura que las palabras largas se ajusten */
  background-color: #ffffff;
  padding: 10px;
  border-radius: 5px;
  width: 100%;
  /* Limita el ancho al 100% del contenedor */
  overflow-x: auto;
  /* Permite el desplazamiento horizontal si es necesario */
}


input,
select {
  padding: 10px;
  font-size: 1.1em;
  background-color: rgb(255, 255, 255);
  border: 1px solid #b6b6b6;
  border-radius: 2px;
  outline: none;
  color: #008cba;
}

/* pre{
  width: 50%;
  padding: 20px;
  background-color: #ffffff;

} */

button {
  background-color: #008CBA;
  color: white;
  padding: 10px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  width: 100%;
  font-size: 1.1em;
}

button:hover {
  background-color: #007B9A;
}
</style>
