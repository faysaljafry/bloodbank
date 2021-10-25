<template>
  <div class="container text-center">
    <h1>Vue Blood Bank</h1>
    <h2>Add your Profile</h2>
    <hr />
    <form @submit.prevent="onsubmit()">
      <div class="row">
        <div class="col-6">
          <div class="form-group">
            <label for="fullName">Full Name</label>
            <input
              type="text"
              id="fullName"
              class=" form-control"
              v-model="userData.fullName"
              required
            />
          </div>
        </div>

        <div class="col-6">
          <div class="form-group">
            <label for="email">Email</label>
            <input
              type="email"
              id="Email"
              class="form-control"
              v-model="userData.email"
              required
              email
              pattern="[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,4}$"
            />
          </div>
        </div>
      </div>

      <div class="row">
        <div class="col-6">
          <div class="form-group">
            <label for="city">City</label>
            <input
              type="text"
              id="city"
              class=" form-control"
              v-model="userData.city"
              required
            />
          </div>
        </div>

        <div class="col-6">
          <div class="form-group">
            <label for="contact">Contact</label>
            <input
              type="text"
              maxlength="11"
              id="contact"
              class="form-control"
              v-model.number="userData.contact"
              required
              pattern="[03][0-9]*"
            />
          </div>
        </div>
      </div>
      <div class="row px-2">
        <label for="bloodGroup">Blood Group</label>
        <select id="bloodGroup" class="form-control" v-model="selectedbg">
          <option v-for="bg in bloodGroup" v-bind:key="bg">
            {{ bg }}
          </option>
        </select>
      </div>
      <hr />
      <div class="row">
        <div class="d-block">
          <button type="submit" class="btn btn-primary" v-if="!isEdit">
            Add Your Profile
          </button>
          <button
            type="button"
            v-if="isEdit"
            @click="makeEdit()"
            class="btn btn-primary"
          >
            Edit Record
          </button>
        </div>
      </div>
    </form>

    <!--This is to show data back to the table-->
    <table id="firstTable" class="table table-dark mt-5">
      <thead>
        <tr>
          <th class="box" @contextmenu="onContextMenu($event, 'fullName')">
            Full Name
          </th>
          <th class="box" @contextmenu="onContextMenu($event, 'email')">
            E-mail
          </th>
          <th class="box" @contextmenu="onContextMenu($event, 'contact')">
            Phone
          </th>
          <th class="box" @contextmenu="onContextMenu($event, 'city')">
            City
          </th>
          <th @contextmenu="$event.preventDefault()">
            Blood Group
          </th>
          <th @contextmenu="$event.preventDefault()">Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for="(row, index) in Profiles"
          v-bind:key="row"
          @contextmenu="modifyOptions($event, index, row)"
        >
          <td>{{ row.fullName }}</td>
          <td>{{ row.email }}</td>
          <td>{{ row.contact }}</td>
          <td>{{ row.city }}</td>
          <td>{{ row.bloodGroup }}</td>
          <td>
            <button
              class="mx-1 btn btn-danger"
              title="Delete"
              @click="deleteRec(index)"
            >
              <svg
                xmlns="http://www.w3.org/2000/svg"
                width="17"
                height="17"
                fill="white"
                class="bi bi-trash-fill"
                viewBox="0 0 16 16"
              >
                <path
                  d="M2.5 1a1 1 0 0 0-1 1v1a1 1 0 0 0 1 1H3v9a2 2 0 0 0 2 2h6a2 2 0 0 0 2-2V4h.5a1 1 0 0 0 1-1V2a1 1 0 0 0-1-1H10a1 1 0 0 0-1-1H7a1 1 0 0 0-1 1H2.5zm3 4a.5.5 0 0 1 .5.5v7a.5.5 0 0 1-1 0v-7a.5.5 0 0 1 .5-.5zM8 5a.5.5 0 0 1 .5.5v7a.5.5 0 0 1-1 0v-7A.5.5 0 0 1 8 5zm3 .5v7a.5.5 0 0 1-1 0v-7a.5.5 0 0 1 1 0z"
                />
              </svg>
            </button>
            <button
              class="btn btn-danger mx-1"
              title="Edit"
              @click="EditRecord(row, index)"
            >
              <svg
                xmlns="http://www.w3.org/2000/svg"
                width="16"
                height="16"
                fill="currentColor"
                class="bi bi-pen-fill"
                viewBox="0 0 16 16"
              >
                <path
                  d="m13.498.795.149-.149a1.207 1.207 0 1 1 1.707 1.708l-.149.148a1.5 1.5 0 0 1-.059 2.059L4.854 14.854a.5.5 0 0 1-.233.131l-4 1a.5.5 0 0 1-.606-.606l1-4a.5.5 0 0 1 .131-.232l9.642-9.642a.5.5 0 0 0-.642.056L6.854 4.854a.5.5 0 1 1-.708-.708L9.44.854A1.5 1.5 0 0 1 11.5.796a1.5 1.5 0 0 1 1.998-.001z"
                />
              </svg>
            </button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import _ from "lodash";
export default {
  data() {
    return {
      userData: {
        fullName: "",
        email: "",
        contact: "",
        city: "",
      },
      bloodGroup: ["A+", "B+", "AB+", "O+", "O-", "AB-"],
      selectedbg: "AB+",
      Profiles: [],
      isEdit: false,
      indextoEdit: -1,
    };
  },
  mounted() {
    if (localStorage.getItem("Profiles")) {
      try {
        this.Profiles = JSON.parse(localStorage.getItem("Profiles"));
      } catch (e) {
        localStorage.removeItem("Profiles");
      }
    }
  },
  components: {},
  methods: {
    modifyOptions(e, index, row) {
      //console.log("The modifier was called", e, index);
      e.preventDefault();
      this.$contextmenu({
        x: e.x,
        y: e.y,
        items: [
          {
            label: "Edit",
            onClick: () => {
              this.EditRecord(row, index);
            },
          },
          {
            label: "Delete",
            onClick: () => {
              this.deleteRec(index);
            },
          },
        ],
      });
    },

    onContextMenu(e, id) {
      //prevent the browser's default menu
      e.preventDefault();
      //shou our menu
      this.$contextmenu({
        x: e.x,
        y: e.y,
        items: [
          {
            label: "Sort Ascending",
            onClick: () => {
              switch (id) {
                case "fullName":
                  this.Profiles = _.orderBy(
                    JSON.parse(JSON.stringify(this.Profiles)),
                    ["fullName"],
                    ["asc"]
                  );
                  break;
                case "email":
                  this.Profiles = _.orderBy(
                    JSON.parse(JSON.stringify(this.Profiles)),
                    ["fullName"],
                    ["asc"]
                  );
                  break;
                case "city":
                  this.Profiles = _.orderBy(
                    JSON.parse(JSON.stringify(this.Profiles)),
                    ["fullName"],
                    ["asc"]
                  );
                  break;
                case "contact":
                  this.Profiles = _.orderBy(
                    JSON.parse(JSON.stringify(this.Profiles)),
                    ["contact"],
                    ["asc"]
                  );
                  break;
              }
            },
          },
          {
            label: "Sort Descending",
            onClick: () => {
              switch (id) {
                case "fullName":
                  this.Profiles = _.orderBy(
                    JSON.parse(JSON.stringify(this.Profiles)),
                    ["fullName"],
                    ["desc"]
                  );
                  break;
                case "email":
                  this.Profiles = _.orderBy(
                    JSON.parse(JSON.stringify(this.Profiles)),
                    ["fullName"],
                    ["desc"]
                  );
                  break;
                case "city":
                  this.Profiles = _.orderBy(
                    JSON.parse(JSON.stringify(this.Profiles)),
                    ["fullName"],
                    ["desc"]
                  );
                  break;
                case "contact":
                  this.Profiles = _.orderBy(
                    JSON.parse(JSON.stringify(this.Profiles)),
                    ["contact"],
                    ["desc"]
                  );
                  break;
              }
            },
          },
        ],
      });
    },
    onsubmit() {
      const record = JSON.parse(JSON.stringify(this.userData));
      // console.log(record);
      var found = this.Profiles.find(
        (element) => element.email == record.email
      );
      if (!found) {
        const toAdd = {
          fullName: this.userData.fullName,
          email: this.userData.email,
          city: this.userData.city,
          contact: this.userData.contact,
          bloodGroup: this.selectedbg,
        };

        this.Profiles.push(toAdd);
        console.log(JSON.parse(JSON.stringify(this.Profiles)));

        localStorage.setItem("Profiles", JSON.stringify(this.Profiles));
        found = false;
        // console.log(this.Profiles);
      } else {
        alert("The Email Already is in the dataBase! Please Retry!");
      }
    },
    deleteRec(id) {
      this.Profiles.splice(id, 1);
      localStorage.setItem("Profiles", JSON.stringify(this.Profiles));
      console.log("The delete was called", id);
      console.log(JSON.parse(JSON.stringify(this.Profiles)));
    },
    EditRecord(row, index) {
      this.isEdit = true;
      this.indextoEdit = index;
      //Populating data with Row Items
      this.userData.fullName = row.fullName;
      this.userData.email = row.email;
      this.userData.city = row.city;
      this.userData.contact = row.contact;
      this.selectedbg = row.bloodGroup;
    },
    makeEdit() {
      var found = this.Profiles.find(
        (element) => element.email == this.userData.email
      );
      if (!found) {
        this.Profiles[this.indextoEdit].email = this.userData.email;
        this.Profiles[this.indextoEdit].fullName = this.userData.fullName;
        this.Profiles[this.indextoEdit].city = this.userData.city;
        this.Profiles[this.indextoEdit].contact = this.userData.contact;
        this.Profiles[this.indextoEdit].bloodGroup = this.selectedbg;
        localStorage.setItem("Profiles", JSON.stringify(this.Profiles));
      } else {
        alert("This email is Already in the List, Please re-write the email!");
      }

      //resetting The forms and turning back the function to add profile again
      this.isEdit = false;
      this.userData.fullName = "";
      this.userData.email = "";
      this.userData.city = "";
      this.userData.contact = "";
      this.selectedbg = "AB+";
    },
  },
};
</script>

<style scoped>
.overflow-hidden {
  overflow: hidden;
}
</style>
