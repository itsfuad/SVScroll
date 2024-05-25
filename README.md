# Scrollbar for svelte apps

This is a simple svelte component that adds a scrollbar to the body of the page. It is useful when you want to hide the scrollbar of the body and add a custom scrollbar to the body.

## Installation

```bash
npm install -D svscroll
```

## Usage

```svelte

<script>
    import Scrollbar from 'svscroll';
</script>

<Scrollbar height="100%" width="100%" zIndex={1}>
    <div>
        <h1>My custom scrollbar</h1>
        <p>This is a custom scrollbar for the body of the page</p>
    </div>
</Scrollbar>
```