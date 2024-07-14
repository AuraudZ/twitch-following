<script lang="ts">
  const client_id = import.meta.env.VITE_CLIENT_ID;
  const redirect_uri = import.meta.env.VITE_REDIRECT_URI;
  let params = `client_id=${client_id}&redirect_uri=${redirect_uri}&response_type=token&scope=user:read:follows`;
  const token = document.location.hash;
  if (token) {
    const accessToken = token.split("=")[1].split("&")[0];
    localStorage.setItem("token", accessToken);
    document.location.hash = "";
    document.location.reload();
  }

  const access_token = localStorage.getItem("token");
  let user_id = "";
  async function getUserId() {
    const res = await fetch("https://api.twitch.tv/helix/users", {
      method: "GET",
      headers: {
        "Client-ID": client_id,
        Authorization: `Bearer ${access_token}`,
      },
    });
    const json = await res.json();
    user_id = json.data[0].id;
    return user_id;
  }

  async function getAllChannels() {
    let cursor = "";
    let dd: any[] = [];
    const res = await fetch(
      `https://api.twitch.tv/helix/channels/followed?user_id=${user_id}&after=${cursor}&first=100`,
      {
        method: "GET",
        headers: {
          "Client-ID": client_id,
          Authorization: `Bearer ${access_token}`,
        },
      }
    );
    const json = await res.json();
    dd.push(json.data);
    cursor = json.pagination.cursor;
    while (cursor) {
      const res = await fetch(
        `https://api.twitch.tv/helix/channels/followed?user_id=${user_id}&after=${cursor}&first=100`,
        {
          method: "GET",
          headers: {
            "Client-ID": client_id,
            Authorization: `Bearer ${access_token}`,
          },
        }
      );
      const json = await res.json();
      dd.push(json.data);
      cursor = json.pagination.cursor;
    }
    const merged = [].concat.apply([], dd);
    return merged;
  }
  let channels: string | any[] = [];

  getUserId().then((data) => {
    user_id = data;
    const dd = getAllChannels();
    dd.then((data) => {
      channels = data;
      console.log(channels);
    });
  });

  function formatTimestamp(timestamp: number) {
    const date = new Date(timestamp);

    const months = [
      "January",
      "February",
      "March",
      "April",
      "May",
      "June",
      "July",
      "August",
      "September",
      "October",
      "November",
      "December",
    ];

    const day = date.getDate();
    const month = months[date.getMonth()];
    const year = date.getFullYear();

    const ordinalSuffix = (day: any) => {
      if (day > 3 && day < 21) return "th";
      switch (day % 10) {
        case 1:
          return "st";
        case 2:
          return "nd";
        case 3:
          return "rd";
        default:
          return "th";
      }
    };

    return `${month} ${day}${ordinalSuffix(day)}, ${year}`;
  }
</script>

<main class="m-5 mb-7">
  <div class="text-center justify-center">
    <a href="https://id.twitch.tv/oauth2/authorize?{params}"
      >Connect with Twitch</a
    >
  </div>
  <div
    class="grid gap-4 grid-cols-1 md:grid-cols-6 sm:grid-cols-3 lg:grid-cols-6"
  >
    {#if channels.length > 0}
      {#each channels as channel}
        <div>
          <a href={`https://twitch.tv/${channel.broadcaster_login}`}>
            <h2>{channel.broadcaster_name}</h2>
          </a>
          <p>{formatTimestamp(Date.parse(channel.followed_at))}</p>
        </div>
      {/each}
    {/if}
  </div>
</main>

<style>
</style>
