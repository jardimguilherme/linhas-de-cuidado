<template>
  <div>
    <breadcrumb/>
    <v-container class="vcontainer">
      <v-row>
        <v-col class="main-col">
          <v-sheet
            v-if="admin"
            class="vsheet pt-3"
            outlined
            max-height="56px"
          >
            <p class="title" >Nova publicação</p>
          </v-sheet>

          <v-container class="forms">
            <v-form v-model="isValid">
              <v-row justify="space-between" style="max-width: 690px">
                  <v-text-field
                    v-model="directory.title"
                    outlined
                    dense
                    label="Título"
                    :rules="inputRules"
                    required
                    style="max-width: 335px; "
                  />
                  <v-text-field
                    v-model="directory.description"
                    outlined
                    dense
                    label="Descrição"
                    :rules="inputRules"
                    required
                    style="max-width: 335px;"
                  />
              </v-row>
              <div v-for="(section) in sections" :key="section.id">
                <v-row>
                  <v-text-field
                    v-model="section.title"
                    outlined
                    dense
                    label="Título da seção"
                    :rules="inputRules"
                    required
                    style="max-width: 335px; margin-top: 1rem"
                  />
                </v-row>
                <v-row>
                  <v-textarea
                    v-model="section.description"
                    outlined
                    dense
                    label="Texto da seção"
                    :rules="inputRules"
                    required
                  />
                </v-row>
              </div>

              <v-row justify="center">
                <v-btn
                  fab
                  small
                  color="primary"
                  @click="addSection"
                >
                <v-icon color="white">mdi-plus</v-icon>
                </v-btn>
              </v-row>
              <v-btn
                class="vbtn"
                color="primary"
                type="submit"
                :disabled="!isValid"
                @click.prevent.stop="addDirectory"
              >
                Salvar
                </v-btn>
            </v-form>
          </v-container>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script>
import api from '../services/api';
import Breadcrumb from '../components/Breadcrumb.vue';

export default {
  components: { Breadcrumb },
  data() {
    return {
      sections: [],
      directory: {},
      isValid: false,
      inputRules: [
        (v) => !!v || 'Campo Obrigatório',
      ],
    };
  },
  computed: {
    dirId() {
      return this.$route.params.id;
    },
    guia() {
      return String(this.$route.params.guia);
    },
    admin() {
      return api.defaults.headers.common.Authorization;
    },
  },
  mounted() {
    this.getDirectory();
    this.getPublications();
  },
  methods: {
    addSection() {
      this.sections.push({ title: '', text: '' });
    },
    async getDirectory() {
      const response = await api.get(`/${this.guia}/${this.dirId}`);
      this.directory = response.data;
    },
    async getPublications() {
      const response = await api.get(`/publications/dir/${this.dirId}`);
      this.sections = response.data;
    },
    async addDirectory() {
      await api.put(`/${this.guia}/${this.dirId}`, {
        title: this.directory.title,
        description: this.directory.description,
      });

      await this.sections.forEach(async (section) => {
        await api.put(`/publications/${section.id}`, {
          title: section.title,
          description: section.description,
          isFromGuide: section.isFromGuide,
          directory: section.directory,
        });
      });

      this.$router.push('/');
    },
  },

};
</script>

<style lang="scss" scoped>

.main-col {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.vcontainer {
  display: flex;
  justify-content: center;
  padding: 0;
  overflow: hidden;
}

.forms {
  padding: 2rem;
}

.vsheet {
  cursor: pointer;
  max-width: 900px;
  width: 100%;
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  border-right: none;
  border-left: none;
  height: 156px;
  overflow: hidden;

}

.vbtn {
  margin: 3rem 1rem 1rem 1rem;
}

.title{
  font-style: normal;
  font-weight: bold;
  font-size: 24px !important;
  line-height: 32px ;
  letter-spacing: 1px !important;
  color: #3988B8;
}

</style>
