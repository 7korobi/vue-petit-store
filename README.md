## Install

```shell
yarn add vue-petit-store
```

## API

### mixin generator

  name | args | description
  :-- | :-- | :--
  replaceState | id | sync vm[id] with URL using history.replaceState
  pushState | id | sync vm[id] with URL using history.pushState
  sessionStorage | id | sync vm[id] with window.sessionStorage
  localStorage | id | sync vm[id] with window.localStorage
  cookie | id | sync vm[id] with document.cookie
  firestore_doc | id, path | sync vm[id] with firestore document at vm[path]
  firestore_collection | id, path, chk, query | sync vm[id] with firestore in query (indexed by doc id)
  firestore_model | id, path | sync memory-orm with firestore document at vm[path]
  firestore_models | id, path, chk, query | sync memory-orm with firestore in query
  vuex_read | path, keys | sync vm[id] with vuex getter only work on update mutation
  vuex | path, keys | sync vm[id] with vuex work on update mutation
  path_by | id, [id1, id2, ...] | (see examples)


## examples

``` javascript
const { replaceState, pushState, sessionStorage, localStorage, cookie,
  firestore_models, firestore_model, firestore_collection, firestore_doc,
  vuex_read, vuex,
} = require("vue-petit-store")
```

### example 1
``` javascript
module.exports = {
  mixins: [
    replaceState("idx"),
    path_by("idx", ["folder", "book", "part", "phase", "chat"]),
  ],
  data () {
    return {
      idx: "a-1"
    }
  }
}
```

  helper method | description
  :-- | :--
  vm.idx_type.by_url | type adjust

  property | description
  :-- | :--
  vm.idx | VueRouter params.idx or query.idx or "a-1"
  vm.idx_default | "a-1"
  vm.idx_a | vm.idx separeted by "-"
  vm.folder_id | "a"
  vm.book_id   | "a-1"
  vm.part_id   | undefined ( because part position 3 )
  vm.phase_id  | undefined ( because part position 4 )
  vm.chat_id   | undefined ( because part position 5 )
  vm.folder | require("memory-orm").Query.folders.find(vm.folder_id)
  vm.book   | require("memory-orm").Query.books.find(vm.book_id)
  vm.part   | require("memory-orm").Query.parts.find(vm.part_id)
  vm.phase  | require("memory-orm").Query.phases.find(vm.phase_id)
  vm.chat   | require("memory-orm").Query.chats.find(vm.chat_id)

### example 2

``` javascript
module.exports = {
  mixins: [
    localStorage("page_by"),
  ],
  data () {
    return {
      page_by: 30
    }
  }
}
```

  helper method | description
  :-- | :--
  vm.page_by_type.by_str | type adjust

  property | description
  :-- | :--
  vm.page_by | window.localStorage.getItem("page_by") or 30
  vm.page_by_default | 30


### example 3

``` javascript
module.exports = {
  mixins: [
    vuex('firebase',['user', 'credential'])
  ],
}
```

  property | description
  :-- | :--
  vm.user | vm.$store.state.firebase.user
  vm.credential | vm.$store.state.firebase.credential
  vm.user = { test: 'test' } | vm.$store.commit("firebase/update", { user: { test: 'test' }})
  vm.credential = { test: 'test' } | vm.$store.commit("firebase/update", { credential: { test: 'test' }})


### example 4

``` javascript
module.exports = {
  mixins: [
    firestore_doc("sign", function() { return this.user && `user/${ this.user.uid }` })
  ],
  data () {
    return {
      user: {
        uid: "TEST_UID",
      },
      sign: {
        sign: "",
        introduction: "",
      }
    }
  }
}
```

  helper method | description
  :-- | :--
  vm.sign_join | join with event handler
  vm.sign_del | delete from snapshot
  vm.sign_add | set doc to snapshot

  property | description
  :-- | :--
  vm.sign | sync with sign_snap
  vm.sign_snap | firestore doc at "user/TEST_UID"
  vm.sign_default | { sign: '', introduction: '' }
  vm.sign_path | "user/TEST_UID"


### example 5

``` javascript
module.exports = {
  mixins: [
    firestore_collection("markers",
      function () { return "marker" }
      function () { return this.uid && this.part_id }
      function (ref) { return ref.where('uid','==', this.uid ).where('part_id','==', this.part_id ) }
  ],
  data () {
    return {
      uid: "TEST_UID"
      part_id: "a-1-1"
      markers: {}
    }
  }
}
```

  helper method | description
  :-- | :--
  vm.marker_join | join with event handler
  vm.markers_del | delete id from snapshot
  vm.markers_add | append doc to snapshot

  property | description
  :-- | :--
  vm.markers | firestore sync here. vm.markers[doc_id] = doc_data
  vm.markers_snap | firestore collection at "marker"
  vm.markers_default | Query
  vm.markers_path | "marker"
  vm.markers_chk | true ("TEST_UID" && "a-1-1")
  vm.markers_query | vm.markers_snap.where('uid','\=\=','TEST_UID').where('part_id','\=\=','a-1-1')


### example 6

``` javascript
module.exports = {
  mixins: [
    firestore_models("markers",
      function () { return "marker" }
      function () { return this.uid && this.part_id }
      function (ref) { return ref.where('uid','==', this.uid ).where('part_id','==', this.part_id ) }
  ],
  data () {
    return {
      uid: "TEST_UID"
      part_id: "a-1-1"
    }
  }
  computed: {
    markers () {
      return this.uid && Query.markers.own(this.uid)
    }
  }
}
```

  helper method | description
  :-- | :--
  vm.marker_join | join with event handler
  vm.markers_del | delete id from snapshot
  vm.markers_add | append doc to snapshot

  property | description
  :-- | :--
  vm.markers | Query.markers.own("TEST_UID")
  vm.markers_snap | firestore collection at "marker"
  vm.markers_default | Query
  vm.markers_path | "marker"
  vm.markers_chk | true ("TEST_UID" && "a-1-1")
  vm.markers_query | vm.markers_snap.where('uid','\=\=','TEST_UID').where('part_id','\=\=','a-1-1')

