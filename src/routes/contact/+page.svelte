<svelte:head>
  <title>Contact me</title>
</svelte:head>
  <nav>
    <a href=".">Home</a>
    <a href="projects">Projects</a>
    <a href="contact" class="current">Contact</a>
    <a href="resume">Resume</a>
    <a href="https://github.com/yamat0002" target="_blank">GitHub</a>
</nav>

    <h1>Contact</h1>

    <form action="mailto:abodayehayat@gmail.com" method="GET">
        <label>Email
            <input type="email" name="email" value="sender@example.com">
        </label>

        <label>Subject
            <input name="subject" value="Hello">
        </label>

        <label>Message
            <textarea name="body">This is a test message.</textarea>
        </label>

        <button>Send</button>
    </form>

