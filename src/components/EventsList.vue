<template>
    <div class="event-list">
      <h2 class="event-title">Events</h2>
      <ul class="list-group">
        <li
          v-for="event in events"
          :key="event.id"
          class="list-group-item"
        >
            <img src={{ event.img }}>
            <p>
            <strong>Title: </strong>
            {{ event.title }}
            </p>
            <span>{{ event.description }}</span>
            <span>{{ event.isFree? 0: event.price }}</span>
            <span>{{ event.yesVotes }}</span>
            <span>{{ event.noVotes }}</span>
            <small class="timestamp">{{ event.date.toDate().toLocaleString() }}</small>

            <button onclick="() => vote(event.id, true, event.yesVotes, event.noVotes)"> Yes </button>
            <button onclick="() => vote(event.id, false)"> No </button>
        </li>
      </ul>
    </div>
  </template>
  
  <script>
  import { db, auth } from "../firebase";
  import { collection, query, onSnapshot, addDoc, updateDoc, doc, where } from "firebase/firestore";
  
  export default {
    data() {
      return {
        events: [],
        votesList: []
      };
    },
    created() {
      this.fetchevents();
    },
    methods: {
      fetchevents() {
        const q = query(collection(db, "events"));
        onSnapshot(q, (snapshot) => {
          this.events = snapshot.docs.map((doc) => ({
            id: doc.id,
            ...doc.data(),
          }));
        });

        console.log(this.events);
      },
      async vote(id, v, yesVotes, noVotes){
        const user = auth.currentUser;
        const votes = query(collection(db, "votes"), where("userId", "==", user.uid));
        onSnapshot(votes, (snapshot) => {
          this.votesList = snapshot.docs.map((doc) => ({
          id: doc.eventId,
          }));
        });
          if(this.votesList.includes(id)){
            prompt('Cannot Vote Again');
          } else {
            await addDoc(collection(db, "votes"), {
              userId: user.uid,
              eventId: id,
              vote: v,
              createdAt: new Date(),
            });
            const eventDocRef = doc(db, "events", id);
            if(v){
              await updateDoc(eventDocRef, {
                yesVotes: yesVotes+1,
              });
            } else {
              await updateDoc(eventDocRef, {
                noVotes: noVotes+1,
              });
            }
          }
      }
    }
  }
  </script>
  
  <style scoped>

.timestamp {
  font-size: 12px;
  color: gray;
  margin-left: 10px;
  display: inline-block;
  vertical-align: middle;
}
  .event-list {
    padding: 20px;
    background: linear-gradient(180deg, #e0f7fa 0%, #80deea 100%);
    border-radius: 10px;
    box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
    min-height: calc(100vh - 60px); 
    display: flex;
    flex-direction: column;
  }
  
  .event-title {
    font-size: 28px;
    color: #007bff;
    text-align: center;
    margin-bottom: 20px;
  }
  
  .list-group {
    list-style-type: none;
    padding: 0;
    flex-grow: 1;
  }
  
  .list-group-item {
    padding: 15px;
    background-color: white;
    border: 1px solid #ddd;
    border-radius: 5px;
    margin-bottom: 10px;
    cursor: pointer;
    transition: background-color 0.3s ease;
  }
  
  .list-group-item:hover {
    background-color: #f1f1f1;
  }
  
  .last-message {
    display: block;
    color: gray;
    font-size: 14px;
    margin-top: 5px;
  }
  
  body {
    margin: 0;
  }
  
  html, body {
    height: 100%;
    overflow: hidden;
  }
  
  #app {
    min-height: 100vh;
    display: flex;
    flex-direction: column;
  }
  </style>
  