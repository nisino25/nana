<template>
  <div id="app">
    <div class="inner">

      <div v-if="currentPage === 'before'">
        <div class="bg-white p-5 rounded-lg shadow-md w-[85%] max-w-[400px] m-auto absolute top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2">

          <div v-if="!roomOption">
            <button @click="roomOption = 'create'; devSkip('create');" class="bg-[#3581B8] text-white  block w-1/2 mx-auto p-[10px] rounded">Create a room</button>
            <hr class="my-3">
            <button @click="roomOption = 'join'; retriveCode(); devSkip('join');" class="bg-[#13563B] text-white block w-1/2 mx-auto p-[10px] rounded">Join a room</button>
          </div>
          <template v-if="roomOption && !roomCode">
            <h2>Type your name</h2>
              <div class="flex items-center mb-3 border border-gray-300 rounded overflow-hidden">
              <input
                type="text"
                class="flex-grow p-2 outline-none"
                v-model="username"
                placeholder="Enter your username"
              >
              <button
                class="p-2 text-gray-600 hover:text-gray-900"
                @click="username = getRandomName()"
              >
                <i class="fa-solid fa-shuffle"></i>
              </button>
            </div>

              <div class="w-[95%] mx-auto grid grid-cols-5 gap-3 justify-between mb-5">
                <div v-for="(avatar, index) in avatars" :key="index" @click="randomString = avatar.randomString" v-html="avatar.avatar"  :style="{ opacity:  randomString !== avatar.randomString ? '0.6' : '1' }"></div>

                <div class="text-3xl text-gray-600 flex justify-center items-center text-center" @click="generateAvatars('female')">
                  <i class="fas fa-sync"></i>
                </div>
              </div>
              <div class="flex items-center mb-2" v-if="roomOption === 'join'">
                <input type="number" v-model="tempRoomcode" placeholder="Type room code" class="flex-grow p-2 border border-gray-300 rounded">
              </div>
              <!-- <button v-if="readyToPlay" @click="randomName()" class="add-button">ランダム</button> -->
              <button @click="roomOption = null" class="bg-[#B83A4B] text-white block mx-auto p-[10px] rounded w-1/2 mb-2">Back</button>
              <button @click="username = getRandomName();" class="bg-[black] text-white block mx-auto p-[10px] rounded mb-2 w-1/2">Random Name</button>
              <button v-if="readyToPlay && roomOption === 'create'" @click="createARoom()" class="bg-[#3581B8] text-white  block  mx-auto p-[10px] rounded w-1/2 mb-2">Create</button>
              <button v-if="tempRoomcode >= 10000 && tempRoomcode <= 99999 && readyToPlay && roomOption === 'join'" @click="joinARoom()" class="bg-[#3581B8] text-white  block mx-auto p-[10px] rounded w-1/2">Join</button>
          </template>

          <template v-if="roomOption && roomCode">
            <h2>Welcome {{ username }}!</h2>
            <p>Room code: <strong class="font-size: 2.5em; color: crimson; margin-right: 5px; font-weight: bold;">{{ roomCode }}</strong></p>
            <hr>
            <template v-for="(player, index) in players" :key="index">
              <div class="player-list flex items-center gap-2 my-2">
                <span>{{index +1}}.</span>
                <div v-html="regenerate(player.randomString)"></div>
                <p>{{ player.name }}</p>
              </div>
            </template>
            <button v-if="players?.length >= 2 && yourPlayer.isHost"  @click="closeTheRoom()" class="bg-[#3581B8] text-white  block  mx-auto p-[10px] rounded w-1/2 mb-2">Close room</button>
          </template>
        </div>
      </div>

      <div v-if="currentPage === 'game'">
        <div class="main-container">
          <div class="area-container">

            <div class="other-players-area">
              <template v-for="otherPlayer in getOtherPlayers()" :key="otherPlayer.name">
                <div class="other-player relative">
                  <PlayerInfo
                    :player="otherPlayer"
                    :isActive="otherPlayer == currentPlayer && onlineStatus == 'playing'"
                    :hand="otherPlayer.hand"
                  />
  
                  <!-- player action buttons (show only when selected) -->
                  <div v-if="selectedPlyaer?.name === otherPlayer.name" class="card-actions flex-col justify-between">
                    <i 
                      class="fas fa-arrow-up bg-green-500 text-white p-2 rounded" 
                      @click="pickCardFrom(otherPlayer.name,'up')">
                    </i>
                    <i 
                      class="fas fa-arrow-down bg-blue-500 text-white p-2 rounded" 
                      @click="pickCardFrom(otherPlayer.name,'down')">
                    </i>
                  </div>
  
                  <!-- Click to select -->
                  <div v-else class="absolute top-0 left-0 w-full h-full cursor-pointer" @click="selectPlayer(otherPlayer)" ></div>
                </div>                 
                
              </template>
            </div>

            <div class="public-area">
              <div class="temp-block">
                <h5>Public Area - #{{ roomCode }} - 
                  <i @click="reloadPage()" class="fa fa-refresh" aria-hidden="true"></i>
                  <i v-if="yourPlayer.isHost" @click="resetGame()" class="ml-1 fa-solid fa-trash"></i></h5>
                <div class="cards-container">
                  <template v-for="(card) in publicPile" :key="card.id">
                    <div 
                      class="gameCard" 
                      :id="'card-'+card?.id" 
                      :class="[
                        { 'flipped': !card.isInRevealedArea, 'opacity-0': card.captured }
                      ]"  
                    >

                      <div class="card-inner">
                        <div class="card-front">
                          <img src="../public/img/card-front.png">
                          <span>{{ card?.number }}</span>
                        </div>
                        <div class="card-back">
                          <img src="../public/img/card-back.png">
                        </div>
                      </div>
                      <!-- Action buttons (show only when selected) -->
                      <div v-if="selectedCard?.id === card.id" class="card-actions flex-col justify-between">
                        <i class="fas bg-green-500 text-white fa-check" @click="confirmCard(card)"></i>
                        <i class="fas bg-red-500 text-white fa-times" @click="cancelSelection"></i>
                      </div>

                      <!-- Click to select -->
                      <div class="absolute top-0 left-0 w-full h-full cursor-pointer" @click="selectCard(card)" v-else></div>
                    </div>
                    
                  </template>
                </div>
              </div>
            </div>


            <div class="personal-info">
                <!-- <div class="my-player"> -->
                <div class="my-player" :class="{ 'player-mask': currentPlayer !== yourPlayer || onlineStatus !== 'playing' }">
                  <PlayerInfo
                    :player="yourPlayer"
                    :isActive="yourPlayer == currentPlayer && onlineStatus == 'playing'"
                    :hand="yourPlayer.hand"
                    :isYourPlayer="true"
                    :currentSelectedPlyaer="selectedPlyaer"
                  />
                  <!-- player action buttons (show only when selected) -->
                  <div v-if="selectedPlyaer?.name === yourPlayer.name" class="card-actions flex-col justify-between">
                      <i 
                        class="fas fa-arrow-up bg-green-500 text-white p-2 rounded" 
                        @click="pickCardFrom(yourPlayer.name,'up')">
                      </i>
                      <i 
                        class="fas fa-arrow-down bg-blue-500 text-white p-2 rounded" 
                        @click="pickCardFrom(yourPlayer.name,'down')">
                      </i>

                  </div>

                  <!-- Click to select -->
                  <div v-else class="absolute top-0 left-0 w-full h-full cursor-pointer" @click="selectPlayer(yourPlayer)" ></div>
                </div>
  
                <div class="revealed-area">
                  <h5 class="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2">{{ gameMessage }}</h5>
                  <template v-for="(card) in revealedCards" :key="card.id">
                    <div class="card-container oveflow-hidden">
                      <div 
                        class="gameCard mx-auto" 
                        :id="'card-'+card?.id"
                        @click="cardTest(card)"
                      >

                        <div class="card-front">
                          <img src="../public/img/card-front.png">
                          <span>{{ card?.number }}</span>
                        </div>
                        <div class="card-back">
                          <img src="../public/img/card-back.png">
                        </div>

                      </div>
                      <span class="text-xs block text-center mt-2">{{ card.location }}</span>
                      </div>
                    </template>
                </div>
            </div>
            
            <div class="personal-area" :class="{ 'player-mask': currentPlayer !== yourPlayer || onlineStatus !== 'playing' }">


              <div class="perosnal-detail">
                  <div class="cards-container">
                    <template v-for="(card) in sortedYourPlayerHand" :key="card.id">
                      <div 
                        class="gameCard" 
                        :id="'card-'+card?.id"
                      >

                        <div class="card-inner">
                          <div class="card-front">
                            <img src="../public/img/card-front.png">
                            <span>{{ card?.number }}</span>
                          </div>
                          <div class="card-back">
                            <img src="../public/img/card-back.png">
                          </div>
                        </div>
                      </div>
                    </template>
    
                  </div>
              </div>


            </div>

          </div>
        </div>
      </div>

    </div>
  </div>
</template>

<script>

import db from './firebase.js';


import { randomNames } from './name.js';
import PlayerInfo from './components/PlayerInfo.vue';
export default {
  components: {
    PlayerInfo,
  },
  data() {
    return {
      developingMode: false,
      // developingMode: true,

      defaultNumber: 2,
      maxPlayerNumber: 6,
      randomNames,


      currentPage: 'before',
      players: [],
      currentPlayerIndex: 0,
      

      deck:[],
      lastSubmitBy: null,
      publicPile: [],

      selectedCard: null,

      tempCards: [],

      selectedPlyaer: null,

      // ----------------
      roomOption: null,
      roomCode: null,
      tempRoomcode: null,
      generalData: null,

      username: null,
      onlineStatus: '',

      processedAudioEvents: [],
      publicAudioEvents: [],

      gameResults: [],
      previousGameResults: [],

      pickedRandomString: null,
      avatars: [],
      randomString: null,

      isCheckingNow: false,
      gameMessage: "",

    };
  },
  computed: {
    readyToPlay() {
      const namePattern = /^[^\s!@#$%^&*(),.?":{}|<>]+$/;

      // Check if the username is valid
      return namePattern.test(this.username) && this.username?.trim() !== '';
    },
    currentPlayer() {
      return this.players[this.currentPlayerIndex];
    },
    
    readyToSubmit() {

      // If the player's picked hands are empty, return false
      if (this.yourPlayerPickedHands.length === 0) return false;

      // --------------------------------------------------------------------

      // If the public pile is empty
      if (this.publicPile.length === 0) {

        // Single card play
        if (this.yourPlayerPickedHands.length === 1) return true;

        // Check if all cards are the same value
        const sameCardCheck = this.yourPlayerPickedHands.every(
            card => card.value === this.yourPlayerPickedHands[0].value
        );
        if (sameCardCheck) return true;

        // Check for sequence (階段)
        if(this.yourPlayerPickedHands.length < 3) return false
        const tempSuit = this.yourPlayerPickedHands[0].suit;
        let tempValue = this.yourPlayerPickedHands[0].value - 1;

        for (let card of this.yourPlayerPickedHands) {
          if (tempSuit !== card.suit || card.value !== ++tempValue) {
              return false;
          }
        }

        return true;
      }

      // --------------------------------------------------------------------
      let wasPreviousRevolution = this.previousCards.length === 4 && this.previousCards.every(card => card.value === this.previousCards[0].value);

      if(wasPreviousRevolution){
        // Check if all cards are the same value
        const sameCardCheck = this.yourPlayerPickedHands.every(
          card => card.value === this.yourPlayerPickedHands[0].value
        );

        if(sameCardCheck){
          // simple case
          const previousValue = this.isRevolutionGoing ? this.previousCards[0].revolutionValue : this.previousCards[0].value;
          const currentValue = this.isRevolutionGoing ? this.yourPlayerPickedHands[0].revolutionValue : this.yourPlayerPickedHands[0].value;
  
          if (currentValue <= previousValue) {
            return false;
          }

          return true
        }

        // stairs

        // revolution
      }

      // --------------------------------------------------------------------

      // Default regular checking

      // Check stairs
      if (this.yourPlayerPickedHands.length >= this.previousCards.length && this.yourPlayerPickedHands.length >= 3) {
          let stairCheck = true;
          const tempSuit = this.yourPlayerPickedHands[0].suit;
          let tempValue = this.isRevolutionGoing ? this.yourPlayerPickedHands[0].revolutionValue + 1 : this.yourPlayerPickedHands[0].value - 1;

          for (let card of this.yourPlayerPickedHands) {
              const cardValue = this.isRevolutionGoing ? card.revolutionValue : card.value;
              if (tempSuit !== card.suit || (this.isRevolutionGoing ? cardValue !== --tempValue : cardValue !== ++tempValue)) {
                  stairCheck = false;
                  break;
              }
          }

          if (stairCheck) return true;
      }



      // Check if all cards are the same value
      const sameCardCheck = this.yourPlayerPickedHands.every(card => card.value === this.yourPlayerPickedHands[0].value);
      if (!sameCardCheck) return false;

      // Check if the length matches the previous cards
      if (this.yourPlayerPickedHands.length < this.previousCards.length && !wasPreviousRevolution) return false;

      // Check if the current cards' value is higher (or lower in revolution) than the previous cards
      const previousValue = this.isRevolutionGoing || this.isTempRevolutionGoing ? this.previousCards[0].revolutionValue : this.previousCards[0].value;
      const currentValue = this.isRevolutionGoing || this.isTempRevolutionGoing ? this.yourPlayerPickedHands[0].revolutionValue : this.yourPlayerPickedHands[0].value;

      if (currentValue <= previousValue) {
        return false;
      }

      return true;
    },

    revealedCards() {
      // Flatten all players' hands and filter only revealed cards
      return this.players
        .flatMap(player => player.hand)       // get all cards from all players
        .filter(card => card?.isInRevealedArea && !card.captured); // only keep revealed cards
    },

    allRevealedCards() {

      const publicRevealed = this.publicPile.filter(card => card?.isInRevealedArea && !card.captured);

      return [...this.revealedCards, ...publicRevealed];
    },

    yourPlayer() {
      return this.players.find(player => player.name === this.username);
    },
    sortedYourPlayerHand() {
      if (!this.yourPlayer || !this.yourPlayer.hand) return [];
      return [...this.yourPlayer.hand]
        .filter(card => !card.isInRevealedArea && !card.captured) // only unrevealed cards
        .sort((a, b) => a.number - b.number);
    },
  },
  methods: {
    sleep(ms) {
      return new Promise(resolve => setTimeout(resolve, ms));
    },
    reloadPage() {
      if (confirm('Are you sure you want to reload the page?')) {
        location.reload();
      }
    },
    resetGame() {
      if (confirm('Are you sure you want to reset the page?')) {
        this.closeTheRoom()
      }
    },
    shuffleArray(array) {
      for (let i = array.length - 1; i > 0; i--) {
          const j = Math.floor(Math.random() * (i + 1));
          [array[i], array[j]] = [array[j], array[i]];
      }
    },
    getRandomName() {
      let randomName;
      do {
        const randomIndex = Math.floor(Math.random() * this.randomNames.length);
        randomName = this.randomNames[randomIndex];
      } while (this.players.some(player => player.name === randomName));

      // this.generateAvatars();

      return randomName;
    },
    
    initializeDeck() {
      // Step 1: Build full deck (1..12, 3 copies each)
      let fullDeck = [];
      let id = 1;
      for (let num = 1; num <= 12; num++) {
        for (let i = 0; i < 3; i++) {
          fullDeck.push({
            id: id++,
            number: num,
            location: 'publicPile', // Temporary location
            isInRevealedArea: false,
            isCaptured: false,
          });
        }
      }

      // Step 2: Adjust deck based on players.length
      let deck = [...fullDeck];
      const numPlayers = this.players?.length;

      if (numPlayers === 2) {
          deck = deck.filter(card => card.number !== 11 && card.number !== 12);
      } else if (numPlayers === 3) {
          deck = deck.filter(card => card.number !== 12);
      }
      // 4 or 5 players → use all cards

      // Step 3: Shuffle deck
      deck = deck.sort(() => Math.random() - 0.5);

      

      // Step 4: Decide cards per player
      let cardsPerPlayer;
      if (numPlayers === 2) cardsPerPlayer = 10;
      else if (numPlayers === 3) cardsPerPlayer = 8;
      else if (numPlayers === 4) cardsPerPlayer = 7;
      else if (numPlayers === 5) cardsPerPlayer = 6;

      // Step 5: Deal cards
      this.players.forEach(player => {
          player.hand = deck.splice(0, cardsPerPlayer).map(card => {
              card.location = player.name; // Update location to player's name
              return card;
          });
      });
      this.publicPile = deck;

    },
    goToGamePage() {
      if (!this.readyToPlay) return;
      this.shuffleArray(this.players)
      // Logic to start the game

      // Reset the current player index
      this.currentPlayerIndex = 0;
      this.currentPage = 'game';

      this.initializeDeck()
    },

    getOtherPlayers() {
      const currentIndex = this.players.findIndex(player => player.name === this.yourPlayer.name);
      if (currentIndex === -1) return this.players; // Return the full list if currentPlayerName is not found

      const before = this.players.slice(0, currentIndex);
      const after = this.players.slice(currentIndex + 1);

      return after.concat(before);
    },

    async goToNextPlayer() {
        console.log('going to the next player-----');
        // Move to the next player, wrapping around if necessary
        this.currentPlayer.isRevealing = false
        this.currentPlayerIndex = (this.currentPlayerIndex + 1) % this.players.length;

    },
    async passToNext(){
      this.goToNextPlayer()
      this.updatingData()
    },
    async submit(){
      this.lastSubmitBy = this.currentPlayer.name

      await this.updatingData()


      // -----------------------------------------
      await this.updatingData()
      await this.sleep(550)

      for(let card of this.deck){
        if(card.location == 'moving'){
          card.location = 'publicArea'
        }
      }

      



      
      await this.updateAudio('card-submit')
      await this.goToNextPlayer()
      await this.updatingData()



    },
    getPileCardLocation(card){
      if(card.location == 'trash'){
        return `
          top: ${card.verticalPosition}%;
          left: ${card.horizontalPosition - 150}%;
          transform: rotate(${card.rotation}deg) translateX(${card.translateX}px);
          zIndex: ${card.zIndex};
        `;
      }
      return `
        top: ${card.verticalPosition}%;
        left: ${card.horizontalPosition}%;
        transform: rotate(${card.rotation}deg) translateX(${card.translateX}px);
        zIndex: ${card.zIndex};
      `;
    },

    flipCard(cardId) {
      console.log('tryna flip');
      const card = this.publicPile?.find(c => c.id === cardId);
      if (card) card.isFliped = true;
    },

    selectCard(card) {
      if(this.yourPlayer !== this.currentPlayer) return
      if(this.revealedCards.length >= 3) return;
      if(card.isInRevealedArea) return;
      if(card.isCaptured) return;
      if(this.isCheckingNow) return;

      this.selectedCard = card
    },
    async confirmCard(card) {
      if(this.yourPlayer !== this.currentPlayer) return;
      if(card.isInRevealedArea) return;
      

      card.isInRevealedArea = true

      await this.submitProcess();
    },
    cancelSelection() {
      this.selectedCard = null
    },

    selectPlayer(player) {
      if(this.yourPlayer !== this.currentPlayer) return;
      if(this.revealedCards.length >= 3) return;
      if(this.isCheckingNow) return;

      this.selectedPlyaer = player
    },

    async pickCardFrom(playerName, direction) {
      // 1. find the player
      const player = this.players.find(p => p.name === playerName);
      if (!player || !player.hand) return;

      // 2. filter unrevealed cards
      const unrevealed = player.hand.filter(card => !card.isInRevealedArea);

      if (unrevealed.length === 0) return; // nothing left to pick

      // 3. sort by number ascending
      const sorted = [...unrevealed].sort((a, b) => a.number - b.number);

      // 4. pick the card
      let pickedCard;
      if (direction === "up") {
        pickedCard = sorted[sorted.length - 1]; // highest
      } else if (direction === "down") {
        pickedCard = sorted[0]; // lowest
      }

      // 5. reveal it
      if (pickedCard) {
        pickedCard.isInRevealedArea = true;
      }

      this.submitProcess()

    },

    async submitProcess() {
      await this.updateAudio('card-submit')
      this.isCheckingNow = true;

      this.selectedCard = null;
      this.selectedPlyaer = null;
      this.updatingData();

      // check all is same number ---------------
      if(this.allRevealedCards.length > 1) await this.sleep(1000);
      this.isCheckingNow = false;


      const allSame = this.allRevealedCards.every(
        card => card.number === this.allRevealedCards[0].number
      );

      if (!allSame) return this.cleanAndProceed();

      // check 3 card or not

      if(this.allRevealedCards.length < 3) return;

      await this.sleep(1000);
      
      console.log('capturing');
      const capturedNum = this.allRevealedCards[0].number
      this.yourPlayer.captured.push(capturedNum);




      // then capture
      
      this.allRevealedCards[2].captured = true
      this.allRevealedCards[1].captured = true
      this.allRevealedCards[0].captured = true

      
      
      // this.updatingData();
      // await this.sleep(3000);



      // // keep the pile location

      // check if the game is over


      // is 7 included then over
      // see if it adds up to 7
      if(this.yourPlayer.captured.includes(7)) return this.updateResult();
      if(this.yourPlayer.captured.length == 3) return this.updateResult();
      
      const capturedA = this.yourPlayer.captured[0]
      const capturedB = this.yourPlayer.captured[1]
      
      if((capturedA + capturedB === 7) || (Math.abs(capturedA - capturedB) === 7)) return this.updateResult();

      // if not then reset then next player
      this.goToNextPlayer();
      this.updatingData();

      


    },

    async cleanAndProceed(){
      this.revealedCards.forEach(card => {
        card.isInRevealedArea = false;
      });

      this.publicPile.forEach(card => {
        card.isInRevealedArea = false;
      });
      await this.updateAudio('card-submit')

      this.goToNextPlayer();
      this.updatingData();
    },

    // ======================================================
    
    generateAvatar() {
      const randomString = Math.random().toString();
      return {
          avatar: window.multiavatar(randomString),
          randomString: randomString
      };
    },
    randomName(){
      this.username = this.getRandomName();
    },
    retriveCode(){
      this.tempRoomcode = localStorage.getItem('latestRoomCode') || 'No room code found'
    },
    async createARoom() {
      if (this.roomCode) return;
      if(!this.username) return;

      let isUnique = false;

      // Generate a unique room code
      while (!isUnique) {
        this.tempRoomcode = Math.floor(10000 + Math.random() * 90000);
        const docRef = db.collection('nana-rooms').doc(`${this.tempRoomcode}`);
        const doc = await docRef.get();
        if (!doc.exists) {
          isUnique = true;
        }
      }

      // console.log(this.roomCode)
      this.roomCode = this.tempRoomcode
      localStorage.setItem('latestRoomCode', this.roomCode)
      
      this.players.push({name:this.username, isHost:true,randomString: this.randomString, captured: []})

      localStorage.userName = this.userName

      // console.log('sending data')
      const ref = db.collection('nana-rooms')
      ref.doc(`${this.roomCode}`).set({
        games: JSON.stringify([{ gameStatus: 'waiting' }]),
        players: this.players,
        onlineStatus: 'waiting',
        deck: this.deck,
        publicAudio: [],
        publicAudioEvents: [],
      })

      // console.log('he')z;
      this.publicAudioEvents = []
      // this.onlineStatus = 'waiting'
      await this.reciveTheData()
    },

    async joinARoom() {

      const docRef = db.collection('nana-rooms').doc(`${this.tempRoomcode}`);

      try {
        const doc = await docRef.get();
        if (doc.exists) {
          if(doc.data().onlineStatus == 'playing') return alert('This room is closed.')

          this.players = doc.data().players;
          this.onlineStatus = doc.data().players;

          if (!this.players.includes(this.username)) this.players.push({name:this.username, isHost:false,randomString:  this.randomString, captured: []});
          this.roomCode = this.tempRoomcode

          await docRef.update({
            players: this.players,
          });
          this.reciveTheData();
        } else {
          console.log('No such document!');
        }
      } catch (error) {
        console.log('Error getting document:', error);
      }
    },

    reciveTheData(){
      
      db.collection("nana-rooms").doc(`${this.roomCode}`)
      .onSnapshot((doc) => {

        this.generalData = doc.data()
        
        // joining room and wait until it closes
        // if(this.currentPage == 'before'){
        this.onlineStatus = doc.data()?.onlineStatus
        this.players = doc.data()?.players

        this.publicAudioEvents = doc.data()?.publicAudioEvents
        if(this.publicAudioEvents?.length > 0){

          this.publicAudioEvents.forEach((event) => {
            if (!this.processedAudioEvents.find(e => e.timeStamp === event.timeStamp)) {
              this.playSound(event.fileName);
              this.processedAudioEvents.push(event);
            }
          });
        }

        if(this.onlineStatus == 'playing' || this.onlineStatus == 'distributing') {
          this.deck = doc.data().deck;
          this.publicPile = doc.data().publicPile;


          this.lastSubmitBy = doc.data()?.lastSubmitBy


          // check if the game is overr


          this.currentPlayerIndex = doc.data().currentPlayerIndex
          this.currentPage = 'game'
          localStorage.setItem('latestRoomCode', null);

          
          this.isRevolutionGoing = doc.data().isRevolutionGoing
          this.isTempRevolutionGoing = doc.data().isTempRevolutionGoing

          this.gameResults = doc.data().gameResults

          

        }
      
      })
    },

    async closeTheRoom(){

      if(this.players.length <2) return

      this.shuffleArray(this.players)

      this.currentPlayerIndex = 0;
      this.currentPage = 'game';
      
      this.publicAudioEvents = []
      await this.updateAudio('card-shuffle')
      
      this.deck = []
      await this.initializeDeck()

      this.previousGameResults = this.gameResults
      this.gameResults = []

      this.onlineStatus = 'playing'
      const ref = db.collection('nana-rooms')
      ref.doc(`${this.roomCode}`).update({
        gameResults: this.gameResults,
        previousGameResults: this.previousGameResults,
        deck: this.deck,
        players: this.players,
        onlineStatus: this.onlineStatus,
        currentPlayerIndex: this.currentPlayerIndex,
        publicAudioEvents: this.publicAudioEvents,
        publicPile: this.publicPile,

      })
    },
    updatingData(){
      const ref = db.collection('nana-rooms')
      ref.doc(`${this.roomCode}`).update({
        publicPile: this.publicPile,
        onlineStatus: this.onlineStatus,
        currentPlayerIndex: this.currentPlayerIndex,
        players: this.players,
      })
    },

    updateResult(){
      const ref = db.collection('nana-rooms')
      this.gameResults.push(this.yourPlayer.name)
      ref.doc(`${this.roomCode}`).update({
        gameResults: this.gameResults,
        players: this.players,
        publicPile: this.publicPile,
      })
    },

    async updateAudio(fileName){
      await this.playSound(fileName);

      this.processedAudioEvents.push({timeStamp: Date.now(),fileName})

      const ref = db.collection('nana-rooms')
      ref.doc(`${this.roomCode}`).update({
        publicAudioEvents : this.processedAudioEvents
      })
    },

    playSound(fileName) {
      try {
        const audio = new Audio(require(`@/assets/sounds/${fileName}.wav`));
        audio.play().catch(error => {
          console.error('Error playing sound:', error);
        });
      } catch (error) {
        console.error('Error loading sound file:', error);
      }
    },

    generateAvatars() {
      this.tempAvatarCode = null;

      this.avatars = [1, 2, 3, 4, 5,6,7,8,9].map(() => {
        const randomString = Math.random().toString();
        return {
          avatar: window.multiavatar(randomString),
          randomString: randomString
        };
      });
    },


    regenerate(randomString) {
      return window.multiavatar(randomString);
    },

    devSkip(mode){
      if(this.developingMode == false) return
      if(mode == 'create'){
        this.createARoom()
      }else if(mode == 'join'){   
        this.joinARoom();
      }
    }

  },
  watch: {
    gameResults(newVal) {
      if (newVal && newVal.length > 0) {
        this.isCheckingNow = true;
        this.sleep(1000);
        alert(`${newVal[0]} won the game`);
        this.gameMessage = "GAME IS OVER"
      }
    }
  },
  async mounted(){
    console.clear()


    this.generateAvatars();
    this.randomString = this.avatars[0].randomString;

    this.deck = []
    this.lastSubmitBy = null
    this.currentPlayerIndex = 0

    this.username = this.getRandomName();
  },
};
</script>

<style>


  :root {

    --game-card-width: 50px;

    --trash-card-width: 60px;
    --trash-card-height: var(--trash-card-width) * 6 / 5;

    /* --game-card-height: calc(var(--game-card-width) * 6 / 5); */
  }

  * {
    touch-action: manipulation;
    -webkit-user-select: none;

    padding: 0;
    margin: 0;
  }

  html, body {
    overflow: hidden;
    height: 100%;
    margin: 0;
    padding: 0;

    font-family: 'Noto Sans JP', sans-serif;
    background: #13563B;
  }

  img{
    max-width: 100%;
  }

  h2 {
    margin: 0 auto 10px;
    font-size: 1.25em;
    text-align: center;
  }

  h5{
    margin-bottom: 5px;
    font-size: .8em;

    width: 100%;
    text-align: center;
  }

  /* --------------------------------- */

  .player-list svg{
    width: 50px;
    height: 50px;
  }
  /* ---------------------------------------- */

  .main-container{
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%,-50%);

    /* background: red; */


    height: 90vh;

    box-sizing: border-box;

    overflow: hidden;

    color: white;

    width: 100%;
    overflow: hidden;
  }

  .area-container{
    height: 100%;
    width: 90vw;

    margin: auto;

    max-width: 425px;
    max-height: 900px;


    display: grid;
    grid-template-rows: calc(18% - 5px) calc(32.5% - 5px)  calc(18% - 5px) calc(29.5% - 5px);
    
    align-content: space-between;
  }

  .area-container > *{
    /* overflow: hidden; */
    background: #4B6F44;

    display: flex;
    align-items: center;
    justify-content: center;

    border-radius: 10px;
  }

  .card-actions {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    height: 85%;

    display: flex;
    flex-wrap: wrap;
    align-items: center;
    /* gap: 10px; */
    font-size: 24px;
    color: white;
  }

  .card-actions i {
    width: 30px;
    text-align: center;
    cursor: pointer;
    padding: 8px;
    border-radius: 50%;
    font-size: 15px;
      
  }



  /* ---------------------------------------- */

  .area-container .other-players-area{
    display: grid;
    grid-template-columns: repeat(4, calc(25% - 5px));
    justify-content: space-between;
    align-items: center;

    width: 100%;
    height: 100%;
    margin: auto;

    padding: 0 10px;

    box-sizing: border-box;

    
  }
  .playerInfo{
    position: relative;
    text-align: center;
    font-size: .7em;
  }

  .playerInfo .player-box{
    position: relative;
    border: 1px solid black;
    transition: border-color 0.5s ease-in-out, box-shadow 0.5s ease-in-out;
  }

  .playerInfo .player-box:before {
    content: '' !important;
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    box-sizing: border-box; /* Ensure the border is included in the element's size */
    pointer-events: none; /* Ensure the pseudo-element doesn't interfere with interactions */
    transition: opacity .5s ease-in; /* Smooth transition for the pseudo-element */
    background-color: rgba(0, 0, 0, 0.5);

    transition: all .3s ease-in-out;
  }

  .playerInfo .player-box.active:before {
    background-color: rgba(0, 0, 0, 0);
    border: 2px solid #DAA520; /* Change the color as needed */
  }

  .playerInfo .name-container {
    display: flex;
    align-items: center;
    justify-content: center;
    height: 1.5em; /* Adjust based on your font-size and line-height */
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap; /* Prevent line breaks */
    text-align: center; /* Center text horizontally */
    padding: 5px 3px;
  }
  .playerInfo .name-container p {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap; /* Prevent line breaks */
    width: 100%; /* Ensure the span takes the full width */
    text-align: center; /* Center text horizontally */
  }
  .playerInfo .player-image-container{
    display: block;
    width: 100%;
    aspect-ratio: 1;
    /* background: red; */
  }
  .playerInfo .player-image-container .temp-image{
    display: block;
    margin: auto;
    /* width: calc(100% - 2px); */
    width: 100%;
    padding: 5px;
    aspect-ratio: 1;

    box-sizing: border-box;

    background: #87A96B;
  }

  .playerInfo span{
    font-size: 2em;
    line-height: 1.25;

    font-weight: bold;

    text-shadow: 2px 2px 5px rgba(0, 0, 0, 0.5);
  }

  /* ---------------------------------------- */

  .cards-container{
    position: relative;
    margin: auto;
    overflow: hidden;

    display: grid;
    grid-template-columns: repeat(5, 1fr);
    justify-content: space-between;
    gap: 5px;
  }

  .public-arena > *{
    padding: 5px;
    background: #4B6F44;
    /* border-radius: 10px; */

    width: 100%;
    height: 100%;
  
  }

  .public-area .detailed-area{
    /* width: 80%; */
    padding: 5px;
    display: grid;
    align-content: space-around;
    justify-content: center;
    box-sizing: border-box;

    text-align: center;

    font-size: .9em;
  }
  

  .public-area .detailed-area .status-badge{
    display: block;
    margin: 0 auto;
    padding: 5px 10px;
    background: #DAA520;
    border-radius: 5px;
    color: black;
    transition: all .5s ease-in-out;

    filter: brightness(1.2);
  }

  .public-area .detailed-area .status-badge.undo-badge{
    filter: brightness(.5);
  }

  .gameCard{
    display: block;
    position: relative;
    border: 1px solid black;
    border-radius: 5px;
    /* position: absolute; */

    overflow: hidden;
    color: black;

    transition: all .3s ease-in-out;

    width:55px;
    aspect-ratio: 10 / 15;
    /* height: 90px; */

    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.25);
  }

  .card-inner {
    width: 100%;
    height: 100%;
    position: relative;
    transform-style: preserve-3d;
    transition: transform 0.5s;
  }

  .gameCard.flipped .card-inner {
    transform: rotateY(-180deg);
  }

  .gameCard img{
    position: absolute;
    width: 100%;
    height: 100%;
  }

  .card-front{
    width: 100%;
    height: 100%;
  }

  .card-front span{
    position: absolute;
    top: 46%;
    left: 50%;
    transform: translate(-50%,-50%);
    color: #FAEBD7;
    font-size: 1.8em;
  }

  .card-front,
  .card-back {
    width: 100%;
    height: 100%;
    position: absolute;
    backface-visibility: hidden;
    /* border: 1px solid #ccc; */
    box-sizing: border-box;
    
    /* border: 1px solid #000; */

    box-shadow: 2px 2px 1px #888888;
  }

  .card-back {
    transform: rotateY(-180deg);
  }

  /* ---------------------------------------- */

  .personal-info{
    display: grid !important;
    grid-template-columns: calc(32.5% - 5px) calc(67.5% - 5px);
    justify-content: space-between;

    background: unset !important;
    border-radius: unset !important;
  }

  .personal-info > *{
    padding: 5px;
    background: #4B6F44;
    border-radius: 10px;

    width: 100%;
    height: 100%;
  
  }

  .revealed-area {
    position: relative;
    margin: auto;
    overflow: hidden;
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    justify-content: space-between;
  }

  /* ---------------------------------------- */

  .personal-area{
    padding: 10px;
    position: relative;
    z-index: 2;
  }
  .my-player{
    position: relative;
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 2;
  }

  .my-player::before, .personal-area::before{
    content: '';
    position: absolute;
    top: 0%;
    left: 0%;

    width: 100%;
    height: 100%;
    background: black;
    opacity: 0;

    z-index: 5;

    border-radius: 10px;

    transition: all .5s ease-in-out;

    pointer-events: none;
  }

  .my-player::before{
    height: 120% !important;
  }

  .my-player::after{
    content: '';
    position: absolute;
    top: 50%;
    left: 0%;

    width: 100%;
    height: 70%;
    background: #4B6F44;
    /* opacity: 0; */

    z-index: -1;
  }


  .player-mask::before{
    opacity: .5 !important;
  }


  .my-player .card-actions{
    height: 75%;
  }

  .player-action-mask{
    position: absolute;
    display: block;
    top: 0%;
    left: 0%;
    width: 100%;
    height: 100%;

    background:black;

    opacity: .5 !important;
    transition: all .5s ease-in-out;
  }


</style>
