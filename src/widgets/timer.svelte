<script lang="ts">
  import { onMount } from 'svelte';

  export let timeIsSet;

  let hours, minutes, seconds;
  let time = 60;
  let timeInterval;

  function getTimeRemaining(endtime) {
    const total = Date.parse(endtime) - Date.parse(new Date());
    const seconds = Math.floor((total / 1000) % 60);
    const minutes = Math.floor((total / 1000 / 60) % 60);
    const hours = Math.floor((total / (1000 * 60 * 60)) % 24);

    return {
      total,
      hours,
      minutes,
      seconds
    };
  }

  const initializeClock = (endtime) => {
    function updateClock() {
      const t = getTimeRemaining(endtime);

      hours = ('0' + t.hours).slice(-2);
      minutes = ('0' + t.minutes).slice(-2);
      seconds = ('0' + t.seconds).slice(-2);

      if (t.total <= 0) {
        clearInterval(timeInterval);
      }
    }

    updateClock();
    timeInterval = setInterval(updateClock, 1000);
  };

  onMount(() => {});

  export const resetTimer = () => {
    time = 60;
    clearInterval(timeInterval);
  };

  export const startTimer = () => {
    const deadline = new Date(Date.parse(new Date()) + 1 * 1 * 1 * time * 1000);
    initializeClock(deadline);

    return Number(time);
  };
</script>

<div style="height: 1.5rem">
  {#if timeIsSet}
    <div>
      <span class="timer">{hours}</span>
      :
      <span class="timer">{minutes}</span>
      :
      <span class="timer">{seconds}</span>
    </div>
  {:else}
    <div class="field is-flex is-align-items-center">
      <div class="control is-small has-icons-left is-flex is-align-items-center">
        <input
          on:input={(e) => {
            time = e.target.value;
          }}
          class="input is-small"
          type="number"
          placeholder="60 sec"
          style="width: 6rem;"
        />
        <span class="icon has-text-success is-left">
          <i class="mdi mdi-18px mdi-clock" />
        </span>
      </div>
    </div>
  {/if}
</div>

<style>
  .timer {
    display: inline-block;
    width: 1rem;
  }
</style>
