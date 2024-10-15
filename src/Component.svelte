<script>
  import { getContext, onMount } from "svelte";

  export let width = 640;
  export let height = 480;
  export let url = '';
  export let apiKey = '';

  const { styleable } = getContext("sdk");
  const component = getContext("component");

  // Get file ID from Google Drive URL
  const getIdFromUrl = (url) => {
    const idWithExtraJunk = url.split('/')[5] ?? '';
    return idWithExtraJunk.split('?')[0] ?? '';
  };

  // Fetch Google Drive file metadata
  const fetchFileMetadata = async (id) => {
    const response = await fetch(`https://www.googleapis.com/drive/v3/files/${id}?fields=webViewLink,webContentLink,mimeType&key=${apiKey}`);
    if (!response.ok) {
      throw new Error('Failed to fetch file metadata');
    }
    return response.json();
  };

  // Generate embed URL for Google Drive file or folder
  const embedUrl = (id, metadata) => {
    const { mimeType, webContentLink } = metadata;

    if (mimeType === 'application/vnd.google-apps.folder') {
      return `https://drive.google.com/embeddedfolderview?id=${id}#list`;
    }

    // For files, use the preview link or web content link for download
    return webContentLink || `https://drive.google.com/file/d/${id}/preview`;
  };

  // Check if the URL is valid
  const checkUrlValid = (urlWithPreview) => {
    return !!(urlWithPreview.includes("https://drive.google.com/drive/folders/")
            || urlWithPreview.includes("https://drive.google.com/file/"));
  };

  let isValid = false;
  let urlAsEmbedLink = '';

  // Fetch and prepare the URL on component mount
  onMount(async () => {
    if (!url) return;

    try {
      isValid = checkUrlValid(url);
      const fileId = getIdFromUrl(url);

      if (isValid) {
        const metadata = await fetchFileMetadata(fileId);
        urlAsEmbedLink = embedUrl(fileId, metadata);
      }
    } catch (error) {
      console.error("Error fetching file metadata:", error);
      isValid = false;
    }
  });
</script>

<div use:styleable={$component.styles}>
  {#if !url}
    <p>Please enter a Google Drive URL.</p>
  {:else if isValid}
    <iframe src={urlAsEmbedLink} width={width} height={height} allow="autoplay"></iframe>
  {:else}
    <p>Error - Please enter a valid Google Drive URL or ensure you have access.</p>
  {/if}
</div>
