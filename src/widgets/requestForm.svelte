<script>
  let start = new Date();
  const tzoffset = new Date().getTimezoneOffset() * 60000;
  const startLocalISOTime = new Date(Date.now() - tzoffset).toISOString().slice(0, 16);
</script>

<div class="field">
  <label class="label">Title</label>
  <div class="control">
    <input class="input" type="text" required id="title" name="title" />
  </div>
</div>

<div class="columns">
  <div class="column">
    <div class="field">
      <label class="label">Start</label>
      <div class="control">
        <input
          class="input"
          type="datetime-local"
          required
          id="start"
          name="start"
          min={startLocalISOTime}
          on:input={(event) => {
            start = event.target.value;
          }}
        />
      </div>
    </div>
  </div>

  <div class="column">
    <div class="field">
      <label class="label">End</label>
      <div class="control">
        <input class="input" type="datetime-local" required id="end" name="end" min={start} />
      </div>
    </div>
  </div>
</div>
