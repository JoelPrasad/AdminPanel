<style scoped>
@import "/node_modules/@suadelabs/vue3-multiselect/dist/vue3-multiselect.css";
</style>
<style>
.multiselect__content-wrapper {
  position: relative;
}
</style>
<template>
  <div>
    <admin-layout>
      <section class="content">
        <div class="container-fluid">
          <div class="row">
            <div class="col-12">
              <div class="card">
                <div class="card-header">
                  <h3 class="card-title">Users</h3>
                  <div class="card-tools">
                    <button
                      type="button"
                      class="btn btn-info text-uppercase"
                      style="letter-spacing: 0.1em"
                      @click="openModal"
                    >
                      Create
                    </button>
                  </div>
                </div>
                <div class="card-body table-responsive p-0">
                  <table class="table table-hover text-nowrap">
                    <thead>
                      <tr>
                        <th class="text capitalize">ID</th>
                        <th class="text capitalize">Name</th>
                        <th class="text capitalize">E-mail</th>
                        <th class="text capitalize">Joined</th>
                        <th class="text capitalize text-right">Actions</th>
                      </tr>
                    </thead>
                    <tbody>
                      <tr v-for="(user, index) in users.data" :key="index">
                        <td>{{ user.id }}</td>
                        <td>{{ user.name }}</td>
                        <td>{{ user.email }}</td>
                        <td>{{ user.created_at }}</td>
                        <td class="text-right">
                          <button
                            class="btn btn-success text-uppercase"
                            style="letter-spacing: 0.1em"
                            @click="editModal(user)"
                          >
                            Edit
                          </button>
                          <button
                            class="btn btn-danger text-uppercase ml-1"
                            style="letter-spacing: 0.1em"
                            @click="deleteUser(user)"
                          >
                            Delete
                          </button>
                        </td>
                      </tr>
                    </tbody>
                  </table>
                </div>
                <div class="card-footer clearfix">
                    <pagination :links="users.links"></pagination>
                </div>
              </div>
            </div>
          </div>
        </div>
      </section>
      <div class="modal fade" id="modal-lg">
        <div class="modal-dialog modal-lg modal-dialog-centered">
          <div class="modal-content">
            <div class="modal-header">
              <h4 class="modal-title">{{ formTitle }}</h4>
              <button
                type="button"
                class="close"
                @click="closeModal"
                aria-label="Close"
              >
                <span aria-hidden="true">&times;</span>
              </button>
            </div>
            <div class="modal-body overflow-hidden">
              <div class="d-flex flex-column h4">
                <span>Preview: <span class="text-capitalize" >{{ form.name }}</span>
                </span>
              </div>
              <div class="card card-primary">
                <form @submit.prevent="checkMode">
                  <div class="card-body">
                    <div class="form-group">
                      <label for="user" class="h4">User Name</label>
                      <input
                        type="text"
                        class="form-control"
                        placeholder="User Name"
                        v-model="form.name"
                        :class="{
                          'is-invalid': form.errors.name,
                        }"
                        autofocus="autofocus"
                        autocomplete="off"
                      />
                    </div>
                    <div
                      class="invalid-feedback mb-3"
                      :class="{
                        'd-block': form.errors.name,
                      }"
                    >
                      {{ form.errors.name }}
                    </div>

                    <div class="form-group">
                      <label for="email" class="h4">Email</label>
                      <input
                        type="text"
                        class="form-control"
                        placeholder="Email Address"
                        v-model="form.email"
                        :class="{
                          'is-invalid': form.errors.email,
                        }"
                        autocomplete="off"
                      />
                    </div>
                    <div
                      class="invalid-feedback mb-3"
                      :class="{
                        'd-block': form.errors.email,
                      }"
                    >
                      {{ form.errors.email }}
                    </div>

                    <div class="form-group" v-if="editMode">
                      <label for="roles" class="h4">Roles</label>
                      <multiselect
                        v-model="form.roles[0]"
                        :options="roleOptions"
                        :multiple="false"
                        :taggable="true"
                        placeholder="Choose new role"
                        @tag="addTag"
                        label="name"
                        track-by="id"
                      ></multiselect>
                    </div>
                    <div
                      class="invalid-feedback"
                      :class="{
                        'd-block': form.errors.roles
                      }"
                    >
                      {{ form.errors.roles }}
                    </div>

                  </div>
                  <div class="modal-footer justify-content-between">
                    <button
                      type="button"
                      class="btn btn-danger text-uppercase"
                      style="letter-spacing: 0.1em"
                      @click="closeModal"
                    >
                      Cancel
                    </button>
                    <button
                      type="submit"
                      class="btn btn-info text-uppercase"
                      style="letter-spacing: 0.1em"
                      :disabled="!form.name || !form.email || form.processong"
                    >
                      {{ btnSubmit }}
                    </button>
                  </div>
                </form>
              </div>
            </div>
          </div>
        </div>
      </div>
    </admin-layout>
  </div>
</template>
<script>
import AdminLayout from "@/Layouts/AdminLayout";
import Pagination from '@/Components/Pagination.vue';
export default {
  props: ['roles', 'users'],
  components: {
    AdminLayout,
    Pagination,
  },
  data() {
    return {
      editedIndex: -1,
      editMode: false,
      form: this.$inertia.form({
        id: '',
        name: '',
        email: '',
        roles: []
      }),
      roleOptions: this.roles,
    }
  },
  computed: {
    formTitle() {
      return this.editedIndex === -1 ? "Create New User" : "Edit Current User";
    },
    btnSubmit() {
      return this.editedIndex === -1 ? "Create" : "Update";
    },
    checkMode() {
      return this.editMode === false ? this.createUser() : this.editUser();
    },
  },
  methods: {
      addTag(newRole) {
          let tag = {
            name: newRole,
          }
          this.roleOptions.push(tag),
          this.form.roles.push(tag)
      },
    editModal(user) {
      this.editMode = true;
      $("#modal-lg").modal("show");
      this.editedIndex = this.users.data.indexOf(user);
      this.form.name = user.name;
      this.form.email = user.email;
      this.form.id = user.id;
      this.form.roles = user.roles
    },
    openModal() {
        this.form.clearErrors();
      this.form.reset();
      this.editedIndex = -1;
      $("#modal-lg").modal("show");
    },
    closeModal() {
      this.editMode = false;
      this.form.reset();
      this.form.clearErrors();
      $("#modal-lg").modal("hide");
    },
    createUser() {
        this.form.post(this.route("admin.users.store"), {
          preserveScroll: true,
          onSuccess: () => {
            this.form.reset();
            this.closeModal();
            Toast.fire({
              icon: "success",
              title: "New User created!",
            });
          },
        });
    },
    editUser() {
       this.form.patch(
         this.route("admin.users.update", this.form.id, this.form),
         {
           preserveScroll: true,
           onSuccess: () => {
             this.form.reset();
             this.closeModal();
             Toast.fire({
               icon: "success",
               title: "User has been updated!",
             });
           },
         }
       );
    },
    deleteUser(user) {
       Swal.fire({
         title: "Are you sure?",
         text: "You won't be able to revert this!",
         icon: "warning",
         showCancelButton: true,
         confirmButtonColor: "#3085d6",
         confirmButtonText: "Yes",
       }).then((result) => {
         if (result.isConfirmed) {
           this.form.delete(this.route("admin.users.destroy", user), {
             preserveScroll: true,
             onSuccess: () => {
               Swal.fire("Deleted!", "User has been deleted.", "success");
             },
           });
         }
       });
    },
  },
};
</script>
