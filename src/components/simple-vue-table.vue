import SimpleVueTable from './simple-vue-table'

export default {
  components: {
    SimpleVueTable
  },
  data () {
    return {
      items: [
        { id: 1, username: "Curran Wong", contact: "16690110 3116", email: "Mauris.eu@volutpat.net" },
        { id: 2, username: "Donovan Lambert", contact: "16670921 6862", email: "ante.dictum.cursus@natoque.ca" },
        { id: 3, username: "Austin Lindsay", contact: "16591004 4485", email: "non.bibendum.sed@dictummi.com" },
        { id: 4, username: "Jerome Velazquez", contact: "16060105 2525", email: "magna@Etiamimperdiet.com" },
        { id: 5, username: "Alden Hudson", contact: "16880710 2754", email: "torquent@enim.org" },
        { id: 6, username: "Gregory Britt", contact: "16260904 9933", email: "nostra.per@velconvallisin.net" },
        { id: 7, username: "Jarrod Mcconnell", contact: "16930717 4434", email: "metus@acarcu.org" },
        { id: 8, username: "Leonard Pitts", contact: "16690125 6773", email: "a.dui.Cras@utaliquam.com" },
        { id: 9, username: "Jamal Sanders", contact: "16070716 6989", email: "Aliquam.erat@odio.edu" },
        { id: 10, username: "Armand Barry", contact: "16170519 4759", email: "non.hendrerit@ipsum.edu" }
      ],

      columns: [
        { name: 'id', text: 'User ID' },
        { name: 'username', text: 'Name' },
        { name: 'contact', text: 'Contact No.' },
        { name: 'email', text: 'Email Address' }
      ]
    }
  }
}
