<script>
  export let targetID;
  export function refresh(){
    if (!targetID) return
    refreshComponent()
  }

  const baseURI = "http://localhost:17763"
  const supportedMIMEs = [
    'image/apng',
    'image/bmp',
    'image/gif',
    'image/x-icon',
    'image/jpeg', 
    'image/png', 
    'image/svg+xml', 
    'image/tiff', 
    'image/webp'
  ]

  let filepath, filename, filesize, fileuri
  let tags = []

  const _fmtByte = (byteSize) => {
    if (byteSize / 1000000 > 1) {
      return `${parseFloat(byteSize/1000000).toFixed(2)}MB`
    } else if (byteSize / 1000 > 1) {
      return `${parseFloat(byteSize/1000).toFixed(2)}KB`
    } else {
      return `${byteSize}B`
    }
  }

  const _isDisplayable = (contentType) => {
    return supportedMIMEs.find((v) => v == contentType)
  }

  const fetchTargetStats = async () => {
		const resp = await axios.get(`${baseURI}/api/targets/${targetID}/fstats`, {
			headers: { "accept": "application/json" }
		})

		return resp.data
  }

  const fetchTargetData = async () => {
		const resp = await axios.get(`${baseURI}/api/targets/${targetID}`, {
			headers: { "accept": "application/json" }
		})

		return resp.data
  }
    
  const refreshComponent = async () => {
    const fstats = await fetchTargetStats(targetID)
    const fdata = await fetchTargetData(targetID)
    
    filepath = fstats.filepath
    filename = fstats.filename
    filesize = fstats.size
    fileuri = _isDisplayable(fstats.contentType) ? `${baseURI}/api/targets/${targetID}/file` : "assets/unsupported-type.png"
    tags = fdata.tags
	}

  // onMount(async () => {
  //   var rc = setInterval(refreshPage, 3000);
	// 	if (targetID > 0) await refreshComponent()
	// })
</script>

<style>
  .image-wrapper{
    width: 100%;
    height: 500px;
    border: 1px solid #ddd;
  }
  .image-wrapper img {
    object-fit: cover;
    min-width: 100%;
    min-height: 100%;
    width: auto;
    height: auto;
    max-width: 100%;
    max-height: 100%;
  }
</style>

<div class="card">
  <div class="card-image p-relative">
    <div class="image-wrapper">
      <img src={fileuri} alt="" class="img-responsive">
    </div>
    <div class="p-absolute" style="right: 0px; top: 0px; padding: 10px;">
    	{#each tags as tag}
        <span class="chip bg-error">to {tag}</span> 
	    {/each}
    </div>
  </div>
</div>

<div class="card" style="margin-top: 5px; padding: 5px 0px 5px 20px;">
  <div class="tile tile-centered">
    <div class="tile-content">
      <div class="tile-title">{filename}</div>
      <small class="tile-subtitle text-gray">{_fmtByte(filesize)} · {filepath}</small>
    </div>
    <div class="tile-action">
      <button class="btn btn-link">
        <i class="icon icon-more-vert"></i>
      </button>
    </div>
  </div>
</div>