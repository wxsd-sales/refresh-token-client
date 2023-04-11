<script lang="ts">
  import { onMount } from 'svelte';
  import { page } from '$app/stores';
  import { slide } from 'svelte/transition';

  import Prism from 'prismjs';
  import theme from 'prism-svelte-renderer/themes/nightOwlLight';
  import 'prism-themes/themes/prism-coy-without-shadows.css';
  import Highlight from 'prism-svelte-renderer';

  import { AUTH_URL } from '$lib/constants';
  import { createMeeting, getAccessToken, getRefreshToken } from '$lib/utils';
  import { tokenResponseStore } from '$lib/store';

  import RequestForm from '../widgets/requestForm.svelte';
  import Timer from '../widgets/timer.svelte';

  export let data;

  let interval = 0;
  let timerRef;
  let timeIsSet = false;
  let submitIsLoading = false;
  let timerInterval;
  let meetingResponse = {};
  let response;
  let displayError = false;
  let error = {};

  $: response = `  
    hostEmail:             ${meetingResponse.hostEmail}
    hostKey:               ${meetingResponse.hostKey}
    start:                 ${meetingResponse.start}
    end:                   ${meetingResponse.end}
    timezone:              ${meetingResponse.timezone}
    title:                 ${meetingResponse.title}
    sipAddress:            ${meetingResponse.sipAddress}
    siteUrl:               ${meetingResponse.siteUrl}
    webLink:               ${meetingResponse.webLink}
    meetingNumber:         ${meetingResponse.meetingNumber}
    password:              ${meetingResponse.password}
    enabledJoinBeforeHost: ${meetingResponse.enabledJoinBeforeHost}
    joinBeforeHostMinutes: ${meetingResponse.joinBeforeHostMinutes}`;

  onMount(async () => {
    if (!$tokenResponseStore) {
      const code = $page.url.searchParams.get('code');
      if (code) {
        $tokenResponseStore = await getAccessToken(code);
      } else {
        window.location.href = AUTH_URL;
      }
    } else {
      if ($tokenResponseStore.access_token_expiration_date < new Date()) {
        $tokenResponseStore = undefined;
        window.location.href = AUTH_URL;
      }
    }
  });
</script>

{#if $tokenResponseStore}
  <section class="section p-2">
    <div class="container" style="max-width: 100%">
      <section class="hero is-info">
        <div class="hero-body">
          <p class="title">Webex Refresh Token Integration</p>
          <p class="subtitle">Where you no longer need to re-login!</p>
        </div>
      </section>
      <section class="section pb-5" style="padding: 2.5rem">
        <div class="tile is-ancestor">
          <div class="tile is-child is-vertical p-0 is-12">
            <div class="columns">
              <div class="column">
                <div class="box notification is-danger">
                  <h1 class="subtitle is-size-4">Webex Meeting API Request</h1>
                  <pre class="requestAPI mb-5">
                      <span class="code">
                        {@html Prism.highlight('https://webexapis.com/v1/meetings', Prism.languages['js'])}
                      </span>
                    </pre>
                  <div class="box is-danger">
                    <form
                      on:submit|preventDefault={async (e) => {
                        submitIsLoading = true;
                        displayError = false;
                        const start = e.target['start'].value;
                        const end = e.target['end'].value;
                        const title = e.target['title'].value;

                        try {
                          meetingResponse = await createMeeting(
                            { title, start, end },
                            $tokenResponseStore.access_token
                          );
                        } catch (e) {
                          displayError = true;
                          console.log(e);
                          const {
                            response: {
                              data: { message, errors, trackingId }
                            }
                          } = e;

                          error.message = message;
                          error.trackingId = trackingId;
                          error.errors = errors;
                        }

                        submitIsLoading = false;
                      }}
                      id="myform"
                      style="padding: 0.5rem; overflow: auto; height: 12rem"
                    >
                      <RequestForm />
                    </form>
                  </div>
                </div>
              </div>
              <div class="column">
                <div class="box notification is-primary">
                  <h1 class="subtitle is-size-4">Webex Meeting API Response</h1>
                  <pre style="background-color: {theme.plain.backgroundColor}" class="py-3">
                    <Highlight {theme} {Prism} text={response} language="javascript" />
                  </pre>
                </div>
              </div>
            </div>
            <div class="tile is-vertical is-12">
              <div class="box">
                <div id="clockdiv" class="is-flex is-justify-content-flex-end">
                  <Timer {timeIsSet} bind:this={timerRef} />
                  {#if timeIsSet}
                    <button
                      class="button ml-3 is-small is-danger"
                      on:click={() => {
                        timerRef.resetTimer();
                        timeIsSet = false;
                        clearInterval(timerInterval);
                      }}
                    >
                      <span class="icon">
                        <i class="mdi mdi-18px mdi-stop-circle-outline" />
                      </span>
                    </button>
                  {:else}
                    <button
                      class="button ml-3 is-small is-success"
                      on:click={async () => {
                        interval = (timerRef.startTimer() + 1) * 1000;
                        $tokenResponseStore = await getRefreshToken($tokenResponseStore.refresh_token);

                        timerInterval = setInterval(async () => {
                          $tokenResponseStore = await getRefreshToken($tokenResponseStore.refresh_token);
                          timerRef.startTimer();
                        }, interval);

                        timeIsSet = true;
                      }}
                    >
                      <span class="icon">
                        <i class="mdi mdi-18px mdi-play-circle-outline" />
                      </span>
                    </button>
                  {/if}
                </div>
                <table class="table is-fullwidth" id="table">
                  <thead>
                    <tr>
                      <th>Property</th>
                      <th>Value</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr>
                      <th>Access Token</th>
                      <td> {$tokenResponseStore.access_token} </td>
                    </tr>
                    <tr>
                      <th>Refresh Token</th>
                      <td> {$tokenResponseStore.refresh_token} </td>
                    </tr>
                    <tr>
                      <th>Access Token Expiration Date</th>
                      <td>{$tokenResponseStore.access_token_expiration_date} </td>
                    </tr>
                    <tr>
                      <th>Refresh Token Expiration Date</th>
                      <td> {$tokenResponseStore.refresh_token_expiration_date} </td>
                    </tr>
                    <tr>
                      <th>Scope</th>
                      <td> {$tokenResponseStore.scope} </td>
                    </tr>
                  </tbody>
                </table>
              </div>
            </div>
          </div>
        </div>
      </section>
      <section class="section pt-0 pb-6 is-flex is-justify-content-flex-end">
        <button
          type="submit"
          form="myform"
          class={'button is-rounded is-medium is-warning'}
          class:is-loading={submitIsLoading}
          id="button"
          >{'Run'}
        </button>
      </section>
    </div>
  </section>

  {#if displayError}
    <div in:slide out:slide class="notification m-2 is-danger" id="notification">
      <button
        on:click={() => {
          displayError = false;
          error = {};
        }}
        class="delete"
      />
      <div class="columns pb-2">
        <div class="column is-3 p-0 is-flex">Message:</div>
        <div class="column p-0 is-9">
          {error.message}
        </div>
      </div>
      <div class="columns pb-2">
        <div class="column is-3 p-0 is-flex">TrackingId:</div>
        <div class="column p-0 is-9">
          {error.trackingId}
        </div>
      </div>
      <div class="columns">
        <div class="column p-0 is-flex is-3">Errors:</div>
        {#each [{ description: 'blu lublub' }] as { description }}
          <div class="column p-0 is-9">{description}</div>
        {/each}
      </div>
    </div>
  {/if}
{/if}

<style>
  td {
    text-overflow: ellipsis;
    overflow: hidden;
  }
  pre {
    border-radius: 0.5rem;
    white-space: inherit !important;
  }
  .code {
    display: flex;
    justify-content: center;
    align-items: center;
    white-space: pre-line;
  }
  .requestAPI {
    display: flex;
    height: 2.5rem;
    padding: 0.75rem;
    margin-bottom: 1rem;
  }
  .code {
    overflow: auto;
    height: 100%;
  }

  #notification {
    width: 40rem;
    display: block;
    position: fixed;
    z-index: 1;
    right: 0;
    bottom: 0;
  }
  .monospace {
    white-space: wrap !important;
    font-family: monospace;
  }
</style>
