<template>
  <div class="container">
    <form @submit.prevent="checkForm" v-if="!this.user.state">
      <div class="form-group user">
        <h2>Персональные данные</h2>
        <div class="form-control user-name">
          <label for="user_name">Имя</label>
          <input id="user_name"
                 class="user_input"
                 type="text"
                 v-model="user.Name"
                 @change="getUpperCase(user.Name, 'user', 'Name')"
                 @blur="validator.user.Name.$touch()"
          />
          <small v-for="error in validator.user.Name.$errors" :key="error">{{ error.$message }}</small>
        </div>
        <div class="form-control user-age">
          <label for="user_age">Возраст</label>
          <input id="user_age"
                 class="user_input"
                 type="text"
                 v-model="user.Age"
                 @blur="validator.user.Age.$touch()"
                 @change="prefixUser(user.Age, 'user', 'Age')"
          />
          <small v-for="error in validator.user.Age.$errors" :key="error">{{ error.$message }}</small>
        </div>
      </div>
      <div class="form-group form-childrens-add">
        <h2 class="childrens-header">Дети (макс.5)</h2>
        <button class="dtn child-add-button"
                type="button"
                v-if="childrensCounter < 5"
                @click="counter"><p class="btn-child-text-plus">+</p> <p class="btn-child-text"> Добавить ребенка</p>
        </button>
      </div>
      <div v-if="childrensCounter !== 0">
        <div class="form-group childrens" v-for="(child, index) in childrens" :key="index">
          <div class="form-control childrens-name">
            <label for="child_name">Имя</label>
            <input id="child_name"
                   class="children_input"
                   type="text"
                   v-model="child.name"
                   @change="getUpperCaseChild(this.childrens.name, index)"
            />
            <small v-for="error in validator.childrens.$each.$response.$errors[index].name"
                   :key="error">{{ error.$message }}
            </small>
          </div>
          <div class="form-control childrens-age">
            <label for="child_age">Возраст</label>
            <input id="child_age"
                   class="children_input"
                   type="text"
                   v-model="child.age"
                   @change="prefix(childrens.age, 'childrens', 'age', index)"
            />
            <small v-for="error in validator.childrens.$each.$response.$errors[index].age"
                   :key="error">{{ error.$message }}
            </small>
          </div>
          <button class="btn delete-btn" type="button" @click="deleteChild(index)">Удалить</button>
        </div>
      </div>
      <button class="submit_btn" type="submit">Сохранить</button>
    </form>
    <preview v-if="this.user.state" :userName="user.Name" :userAge="user.Age" :children="childrens"
             :userPrefix="user.prefix"></preview>
  </div>
</template>

<script>
import useVuelidate from '@vuelidate/core'
import {required, alpha, numeric, maxLength, helpers} from '@vuelidate/validators'
import {reactive} from 'vue'
import Preview from "@/components/Preview";

export default {
  components: {
    'preview': Preview,
  },
  setup() {
    let user = reactive({
      Name: '',
      Age: '',
      prefix: '',
      state: false
    });
    const childId = 0;
    const childrens = reactive([]);
    const childrensCounter = 0;


    const rules = {
      user: {
        Name: {
          required: helpers.withMessage('Поле обязательно для заполнения', required),
          alpha: helpers.withMessage('Недопустимый формат имени', val => /^[а-яё]*$/i.test(val)),
        },
        Age: {
          required: helpers.withMessage('Поле обязательно для заполнения.', required),
          numeric: helpers.withMessage('Поле должно содержать только цифры.', numeric),
          maxLength: helpers.withMessage('Слишком большой возраст.', maxLength(3))
        }
      },
      childrens: {
        $each: helpers.forEach({
          name: {
            alpha: helpers.withMessage('Недопустимый формат имени.', val => /^[а-яё]*$/i.test(val)),
          },
          age: {
            numeric: helpers.withMessage('Поле должно содержать только цифры. ', numeric),
            maxLength: helpers.withMessage('Слишком большой возраст.', maxLength(2))
          },
        })
      }

    };
    const validator = useVuelidate(
        rules,
        {user, childrens,}
    );
    return {
      user, childrens, validator, childrensCounter, childId
    }
  },

  methods: {
    prefix(value, path, subPath, index) {
      let pref = ['год', 'года', 'лет']
      let n = this[path][index][subPath]
      n = Math.abs(n) % 100;
      let n1 = n % 10;
      if (n > 10 && n < 20) {
        this[path][index].prefix = pref[2];
      } else if (n1 > 1 && n1 < 5) {
        this[path][index].prefix = pref[1];
      } else if (n1 === 1) {
        this[path][index].prefix = pref[0];
      } else {
        this[path][index].prefix = pref[2];
      }
    },
    prefixUser(value, path, subPath) {
      let pref = ['год', 'года', 'лет']
      let n = this[path][subPath]
      n = Math.abs(n) % 100;
      let n1 = n % 10;
      if (n > 10 && n < 20) {
        this[path].prefix = pref[2];
      } else if (n1 > 1 && n1 < 5) {
        this[path].prefix = pref[1];
      } else if (n1 === 1) {
        this[path].prefix = pref[0];
      } else {
        this[path].prefix = pref[2];
      }
    },
    counter() {
      this.childrensCounter++
      this.addChild()
    },
    addChild() {
      this.childId++
      this.childrens.push({name: '', age: '', id: this.childId, prefix: ''})
    },
    deleteChild(index) {
      this.childrens.splice(index, 1)
      this.childrensCounter--
    },
    getUpperCase(value, path, subPath) {
      this[path][subPath] = this[path][subPath][0].toUpperCase() + this[path][subPath].slice(1);
    },
    getUpperCaseChild(value, index) {
      this.childrens[index].name = this.childrens[index].name[0].toUpperCase() + this.childrens[index].name.slice(1);
    },
    checkForm() {
      if (this.validator.$invalid) {
        this.validator.$touch()
        return
      } else {
        this.user.state = true
        console.log(this.user.state)
      }
    }
  }
}
</script>

<style>

</style>
