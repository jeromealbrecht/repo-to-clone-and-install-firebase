<!-- Composant de messagerie -->
<script>
  import { onMount } from 'svelte';
  import { auth, database } from '../../Firebase';
  import { collection, addDoc, getDocs, query, orderBy, onSnapshot } from 'firebase/firestore';

  /**
 * @type {any[]}
 */
  let messages = [];
  let newMessage = '';

  const messagesCollection = collection(database, 'messages');

  onMount(() => {
      if (auth.currentUser) {
          const conversationId = generateConversationId(auth.currentUser.uid, 'userId2');
          const Usermail = generateConversationId(auth.currentUser.email, 'userMail');
          const messagesQuery = query(messagesCollection, orderBy('timestamp'));

          const unsubscribe = onSnapshot(messagesQuery, (snapshot) => {
              messages = snapshot.docs.map(doc => doc.data());
          });

          return () => unsubscribe();
      }
  });

  function sendMessage() {
      if (auth.currentUser) {
          const conversationId = generateConversationId(auth.currentUser.uid, 'userId2');
          const Usermail = generateConversationId(auth.currentUser.email, 'userMail');
          
          // Envoie le nouveau message à la base de données
          addDoc(messagesCollection, {
              sender: auth.currentUser.email,
              content: newMessage,
              timestamp: new Date(),
          });
      }

      // Efface le champ de saisie après l'envoi du message
      newMessage = '';
  }

  // Génère un identifiant unique pour la conversation
  /**
 * @param {any} userId1
 * @param {string} userId2
 */
  function generateConversationId(userId1, userId2) {
      const sortedUserIds = [userId1, userId2].sort();
      return sortedUserIds.join('_');
  }
</script>

<!-- Interface utilisateur -->
<section>
  {#each messages as { avatar, sender, content, timestamp }}
      <div class="wrapper">
          {#if avatar}
              <img src={avatar} alt="Avatar" class="avatar" />
          {:else}
              <img src="https://image.noelshack.com/fichiers/2023/50/4/1702591430-0cb27b4c-2b3f-467e-ba52-0ee8211cf67a.jpg" alt="Default Avatar" class="avatar" />
          {/if}
          <div class="content">
              <span class="spancolor">{sender}</span>
              <hr class="hr-style">
              <span class="spancolor">{content}</span>
          </div>
      </div>
  {/each}

  <h2>Saisissez votre message</h2>
  <textarea bind:value={newMessage} placeholder="Saisissez votre message"></textarea>
  <button on:click={sendMessage}>Envoyer</button>
</section>

<style>
  section {
      width: 100%;
      display: flex;
      flex-direction: column;
      align-items: center;
  }

  .wrapper, textarea {
      display: flex;
      width: 80%;
      background-color: #2e3238;
      color: #fff;
      padding: 2%;
      margin-top: 1rem;
      border-radius: 1rem;
  }

  .avatar {
      width: 50px;
      height: 50px;
      border-radius: 50%;
      margin-right: 10px;
  }
</style>
