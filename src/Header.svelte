<script>
  import { username, user } from './user';

  function signout() {
    user.leave();
    username.set('');
  }
</script>

<header>
<h1>🔫 Chat App</h1>
  {#if $username}
    <div class="user-bio">
      
      <span class="welcome" >Hello <strong>{$username}</strong></span>
      <img src={`https://avatars.dicebear.com/api/human/${$username}.svg`} alt="avatar" /> 
    </div>

    <button class="signout-button" on:click={signout}>Sign Out</button>
  {/if}
</header>