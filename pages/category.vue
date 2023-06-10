<template>
  <div>
    <v-card>
      <v-toolbar color="transparent">
        <v-toolbar-title class="font-weight-medium">
          จัดการหมวดหมู่รายวิชา
        </v-toolbar-title>
        <v-spacer></v-spacer>
      </v-toolbar>
      <v-divider></v-divider>
      <div class="pa-5">
        <v-data-table
          :headers="headers"
          :items="users"
          :search="search"
          :items-per-page="5"
          class="elevation-1 text-no-wrap"
        >
          <template v-slot:top>
            <v-toolbar flat>
              <v-text-field
                v-model="search"
                label="ค้นหา"
                class="ps-4"
                density="compact"
                variant="solo"
                append-inner-icon="mdi-magnify"
                single-line
                hide-details
              ></v-text-field>
              <!-- </v-card-text> -->

              <v-divider class="mx-4" inset vertical></v-divider>
              <v-spacer></v-spacer>
              <v-dialog v-model="dialog" max-width="500px">
                <template v-slot:activator="{ props }">
                  <v-btn
                    class="text-none text-subtitle-1"
                    color="success"
                    size="small"
                    variant="flat"
                    @click="addItems()"
                  >
                    เพิ่มรายวิชา
                  </v-btn>
                </template>

                <v-card>
                  <v-card-title>
                    <span class="text-h6">{{ formTitle }}</span>
                  </v-card-title>

                  <v-card-text>
                    <v-container>
                      <v-row>
                        <v-col cols="12" sm="8" md="8">
                          <v-text-field
                            variant="outlined"
                            v-model="editedItem.ctName"
                            label="รายวิชา"
                            small
                          ></v-text-field>
                        </v-col>
                      </v-row>
                    </v-container>
                  </v-card-text>

                  <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn
                      color="blue-darken-1"
                      variant="text"
                      @click="dialog = false"
                    >
                      ยกเลิก
                    </v-btn>
                    <v-btn color="blue-darken-1" variant="text" @click="save">
                      บันทึก
                    </v-btn>
                  </v-card-actions>
                </v-card>
              </v-dialog>
              <v-dialog v-model="dialogDelete" max-width="390px">
                <v-card>
                  <p class="text-sm-h6 mt-4 text-center">ลบรายวิชาหรือไม่</p>
                  <v-card-actions class="pb-2">
                    <v-spacer></v-spacer>
                    <v-btn
                      color="blue-darken-1"
                      variant="text"
                      @click="dialogDelete = false"
                      >ยกเลิก</v-btn
                    >
                    <v-btn
                      color="blue-darken-1"
                      variant="text"
                      @click="deleteItemConfirm"
                      >ตกลง</v-btn
                    >
                    <v-spacer></v-spacer>
                  </v-card-actions>
                </v-card>
              </v-dialog>
            </v-toolbar>
          </template>
          <template v-slot:item.actions="{ item }">
            <v-icon size="small" class="me-2" @click="editItem(item.raw)">
              mdi-pencil
            </v-icon>
            <v-icon size="small" @click="deleteItem(item.raw)">
              mdi-delete
            </v-icon>
          </template>
          <template v-slot:no-data>
            <v-alert prominent type="error" variant="outlined">
              ไม่พบข้อมูล
            </v-alert>
          </template>
        </v-data-table>
      </div>
    </v-card>
    <v-snackbar v-model="snackbar" timeout="2000" top>
      {{ snackTitle }}

      <template v-slot:actions>
        <v-btn color="blue" variant="text" @click="snackbar = false">
          ปิด
        </v-btn>
      </template>
    </v-snackbar>
  </div>
</template>
<script setup>
import { VDataTable } from "vuetify/labs/VDataTable";
</script>
<script>
export default {
  data() {
    return {
      users: [],
      dialog: false,
      snackTitle: "",
      snackbar: false,
      dialogDelete: false,
      search: "",
      editedItem: {
        ctId: "",
        ctName: "",
        docId: "",
        sequence: "",
      },
      editedIndex: -1,
      headers: [
        { title: "ลำดับ", key: "ctId" },
        { title: "ชื่อรายวิชา", key: "ctName" },
        { title: "จัดการ", key: "actions", sortable: false },
      ],
    };
  },
  async mounted() {
    const useRuntimeConfig = this.$config.public.apiBase;
    this.users = await $fetch(`${useRuntimeConfig}/category/getcategory`).then(
      (res) => {
        return res.data.map((item) => {
          return {
            ctId: item.ctId,
            ctName: item.ctName,
            docId: item.docId,
            sequence: item.sequence,
            search: "",
          };
        });
      }
    );
  },
  methods: {
    async refreshData() {
      const useRuntimeConfig = this.$config.public.apiBase;

      let jsonData = await $fetch(
        `${useRuntimeConfig}/category/getcategory`
      ).then((res) => {
        return res.data.map((item) => {
          return {
            ctId: item.ctId,
            ctName: item.ctName,
            docId: item.docId,
            sequence: item.sequence,
          };
        });
      });
      this.users = jsonData;
    },
    editItem(item) {
      this.editedIndex = this.users.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialog = true;
    },
    addItems() {
      this.editedIndex = -1;
      this.editedItem = {
        ctId: "",
        ctName: "",
        docId: "",
        sequence: "",
      };
      this.dialog = true;
    },
    deleteItem(item) {
      this.editedIndex = this.users.indexOf(item);
      this.editedItem = Object.assign({}, item);
      this.dialogDelete = true;
    },
    deleteItemConfirm() {
      const useRuntimeConfig = this.$config.public.apiBase;
      let json = {
        Id: this.editedItem.ctId,
      };
      $fetch(`${useRuntimeConfig}/category/deletecategory`, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(json),
      }).then((res) => {
        this.dialogDelete = false;
        this.snackbar = true;
        this.snackTitle = "ลบข้อมูลสำเร็จ";
        this.refreshData();
      });
    },
    async save() {
      if (this.editedIndex > -1) {
        let json = {
          name: this.editedItem.ctName,
          sequence: this.editedItem.sequence,
        };
        const useRuntimeConfig = this.$config.public.apiBase;
        await $fetch(
          `${useRuntimeConfig}/category/updatecategory/${this.editedItem.ctId}`,
          {
            method: "PUT",
            headers: {
              "Content-Type": "application/json",
            },
            body: JSON.stringify(json),
          }
        ).then((res) => {
          this.dialog = false;
          this.snackbar = true;
          this.snackTitle = "แก้ไขข้อมูลสำเร็จ";
          this.refreshData();
        });
      } else {
        // check empty data
        if (this.editedItem.ctName == "") {
          this.snackbar = true;
          this.snackTitle = "กรุณากรอกข้อมูลให้ครบถ้วน";
          return;
        }
        let json = {
          name: this.editedItem.ctName,
          sequence: this.users.length + 1,
        };
        const useRuntimeConfig = this.$config.public.apiBase;
        await $fetch(`${useRuntimeConfig}/category/insertcategory`, {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify(json),
        }).then((res) => {
          this.dialog = false;
          this.snackbar = true;
          this.snackTitle = "เพิ่มข้อมูลสำเร็จ";
          this.refreshData();
          console.log(res);
        });
      }
    },
  },
  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "เพิ่มรายวิชา" : "แก้ไขรายวิชา";
    },
  },
  watch: {
    dialog(val) {
      console.log(val);
    },
  },
};
</script>

<style lang="scss" scoped></style>
