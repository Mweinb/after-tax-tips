<script lang="ts">
  const ENDPOINT = 'https://formspree.io/f/xgaakbel';

  let email = $state('');
  let message = $state('');
  let status = $state<'idle' | 'sending' | 'sent' | 'error'>('idle');
  let error = $state('');

  async function submit(e: SubmitEvent) {
    e.preventDefault();
    if (!message.trim()) return;
    status = 'sending';
    error = '';
    try {
      const res = await fetch(ENDPOINT, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json', Accept: 'application/json' },
        body: JSON.stringify({ email: email.trim() || undefined, message, page: window.location.hash }),
      });
      if (!res.ok) {
        const data = await res.json().catch(() => null);
        throw new Error(data?.errors?.map((x: { message: string }) => x.message).join(', ') || `HTTP ${res.status}`);
      }
      status = 'sent';
      email = '';
      message = '';
    } catch (err) {
      status = 'error';
      error = err instanceof Error ? err.message : String(err);
    }
  }
</script>

<h2>Leave Feedback</h2>
{#if status === 'sent'}
  <p>Thanks! Your feedback has been sent.</p>
{:else}
  <form onsubmit={submit}>
    <label for="feedback-email">Email (optional, if you'd like a reply)</label>
    <input id="feedback-email" type="email" bind:value={email} placeholder="you@example.com" />

    <label for="feedback-message" style="margin-top: 1rem;">Message</label>
    <textarea id="feedback-message" rows="6" bind:value={message} required></textarea>

    {#if status === 'error'}
      <div class="notice notice-error" style="margin-top: 1rem;">Could not send feedback ({error}).</div>
    {/if}

    <button type="submit" class="btn btn-primary" style="margin-top: 1rem;" disabled={status === 'sending' || !message.trim()}>
      {status === 'sending' ? 'Sending...' : 'Send'}
    </button>
  </form>
{/if}

<style>
  textarea {
    resize: vertical;
  }
</style>
