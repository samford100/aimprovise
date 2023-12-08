<script>
	export let text
	export let id
	export let onClick
</script>

<button 
  on:click={onClick} 
  id={id}
  class="action-button w-40 h-16 bg--500 rounded-lg cursor-pointer select-none
  active:translate-y-2  
  active:border-b-[0px]
  transition-all duration-150 
  border-b-[0px]  text-black font-bold text-lg mx-1 my-3">
  {text}
</button>