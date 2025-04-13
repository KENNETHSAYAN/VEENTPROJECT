<script lang="ts">
	import { registrantStore } from '$lib/stores';
	import { Dialog } from 'bits-ui';
    import ScheduleDatePicker from './ScheduleDatePicker.svelte';
	import type { GuestProps } from '$lib/types';
	import { guestList } from '$lib/stores/data';
  
	// State management using $state
	let state = $state({
	  showSchedule: false,
	  message: '',
	  guestSearchQuery: '',
	  scheduleDate: null,
	  scheduleTime: '',
	  timezone: 'GMT+08:00 Manila',
	  showGuestSearch: false,
	  selectedGuests: [] as GuestProps[],
	  searchResults: [] as GuestProps[]
	});
	
	// Initialize store with guestList data
	$effect(() => {
	  registrantStore.set(guestList);
	});
  
	// Derived state for filtered guests that updates when query changes
	let filteredGuests = $derived(() => searchGuests(state.guestSearchQuery));
  
	// Functions
	function toggleSchedule() {
	  state.showSchedule = !state.showSchedule;
	}
  
	function toggleGuestSearch() {
	  state.showGuestSearch = !state.showGuestSearch;
	  // Clear previous search results when opening/closing search
	  if (!state.showGuestSearch) {
		state.searchResults = [];
	  }
	}
  
	function selectGuest(guest: GuestProps) {
	  if (!state.selectedGuests.find((g) => g.id === guest.id)) {
		state.selectedGuests = [...state.selectedGuests, guest];
	  }
	  state.guestSearchQuery = '';
	  state.showGuestSearch = false;
	}
  
	function removeGuest(guestId: string) {
	  state.selectedGuests = state.selectedGuests.filter((g) => g.id !== guestId);
	}
  
	async function sendBlast() {
	  if (state.selectedGuests.length === 0) {
		alert('Please select at least one recipient');
		return;
	  }
  
	  if (!state.message.trim()) {
		alert('Please enter a message');
		return;
	  }
  
	  if (state.showSchedule && (!state.scheduleDate || !state.scheduleTime)) {
		alert('Please select both date and time for scheduling');
		return;
	  }
  
	  const blastData = {
		recipients: state.selectedGuests.map((g) => g.id),
		message: state.message,
		scheduleDate: state.showSchedule ? state.scheduleDate : null,
		scheduleTime: state.showSchedule ? state.scheduleTime : null,
		timezone: state.showSchedule ? state.timezone : null,
	  };
  
	  console.log('Sending blast:', blastData);
	}
  
	// Enhanced search function that uses the store data
	function searchGuests(query: string): GuestProps[] {
	  if (!query || query.trim() === '') {
		state.searchResults = $registrantStore; // Show all guests if query is empty
		return state.searchResults;
	  }
	  
	  const lowerCaseQuery = query.toLowerCase();
	  const results = $registrantStore.filter(
		(guest) =>
		  guest.name.toLowerCase().includes(lowerCaseQuery) ||
		  guest.email.toLowerCase().includes(lowerCaseQuery)
	  );
	  
	  state.searchResults = results; // Update search results state
	  return results;
	}
	
	// Function to handle search input changes
	function handleSearchInput() {
	  if (state.guestSearchQuery.length > 0) {
		state.showGuestSearch = true;
	  }
	}
	
	// Effect to update search results when query changes
	$effect(() => {
	  if (state.guestSearchQuery) {
		searchGuests(state.guestSearchQuery);
	  }
	});
  </script>
  
  <div class="p-4 sm:p-6">
	<div class="flex w-full items-center justify-between text-lg font-semibold tracking-tight mb-4 sm:mb-6">
	  <div class="flex items-center gap-2 sm:gap-3">
		<div class="flex h-8 w-8 items-center justify-center rounded-full bg-gray-100 sm:h-10 sm:w-10">
		  <i class="fa-solid fa-envelope text-gray-500"></i>
		</div>
		<div>
		  <h2 class="text-lg font-semibold text-gray-800 sm:text-xl">Send Blast</h2>
		  <p class="mt-0.5 text-xs text-gray-500 sm:mt-1 sm:text-sm">
			Guests will receive the blast via email, SMS or in-app notification.
		  </p>
		</div>
		<Dialog.Close class="cursor-pointer p-2 text-gray-500">
		  <i class="fa-solid fa-xmark"></i>
		</Dialog.Close>
	  </div>
	</div>
  
	<form class="space-y-5">
	  <div class="space-y-2">
		<label for="recipients" class="block text-sm font-medium text-gray-700">Recipients</label>
		<div class="relative">
		  <input
			type="text"
			id="recipients"
			placeholder="Search guests by name or email..."
			bind:value={state.guestSearchQuery}
			onfocus={toggleGuestSearch}
			oninput={handleSearchInput}
			class="w-full rounded-lg border border-gray-300 bg-white p-2 pl-9 text-sm shadow-sm outline-none sm:p-2.5 sm:pl-10 focus:border-blue-500 focus:ring-1 focus:ring-blue-500"
		  />
		  <i class="fa-solid fa-magnifying-glass absolute left-2.5 top-2.5 text-gray-400 sm:left-3 sm:top-3"></i>
		
		  <!-- Search Results Dropdown -->
		  {#if state.showGuestSearch && state.guestSearchQuery.length > 0}
			<div class="absolute z-10 mt-1 w-full rounded-lg border border-gray-300 bg-white shadow-lg">
			  <div class="sticky top-0 bg-gray-50 p-2 border-b border-gray-200 text-xs text-gray-500 flex items-center justify-between">
				<span>Found {state.searchResults.length} guests</span>
				<button 
                  aria-labelledby="close-search"
				  type="button" 
				  class="text-gray-500 hover:text-gray-700"
				  onclick={toggleGuestSearch}
				>
				  <i class="fa-solid fa-times"></i>
				</button>
			  </div>
			  <div class="max-h-48 overflow-y-auto">
				{#if state.searchResults.length === 0}
				  <div class="p-3 text-sm text-gray-500 text-center">
					No matching guests found
				  </div>
				{:else}
				  {#each state.searchResults as guest}
					<button
					  type="button"
					  class="flex w-full cursor-pointer items-center gap-2 px-3 py-2 text-left text-sm hover:bg-gray-100 sm:gap-3 sm:px-4"
					  onclick={() => selectGuest(guest)}
					>
					  <img src={guest.avatar || '/placeholder-avatar.png'} alt={guest.name} class="h-6 w-6 rounded-full sm:h-8 sm:w-8" />
					  <div>
						<div class="text-sm font-medium">{guest.name}</div>
						<div class="text-xs text-gray-500">{guest.email}</div>
					  </div>
					</button>
				  {/each}
				{/if}
			  </div>
			</div>
		  {/if}
		</div>
  
		<!-- Selected Guests Tags -->
		{#if state.selectedGuests.length > 0}
		  <div class="mt-2 flex flex-wrap gap-2">
			{#each state.selectedGuests as guest}
			  <div class="flex items-center gap-1 rounded-full bg-gray-100 px-3 py-1 text-sm">
				<img src={guest.avatar || '/placeholder-avatar.png'} alt={guest.name} class="h-5 w-5 rounded-full" />
				<span>{guest.name}</span>
				<button
                  aria-labelledby="remove-guest"
				  type="button"
				  class="ml-1 text-gray-500 hover:text-gray-700"
				  onclick={() => removeGuest(guest.id)}
				>
				  <i class="fa-solid fa-times"></i>
				</button>
			  </div>
			{/each}
			{#if state.selectedGuests.length > 1}
			  <button 
				type="button"
				class="text-xs text-blue-600 hover:text-blue-800"
				onclick={() => state.selectedGuests = []}
			  >
				Clear all
			  </button>
			{/if}
		  </div>
		{/if}
	  </div>
  
	  <div class="space-y-2">
		<label for="message" class="block text-sm font-medium text-gray-700">Message</label>
		<textarea
		  id="message"
		  bind:value={state.message}
		  rows="4"
		  class="w-full resize-none rounded-lg border border-gray-300 bg-white p-2 text-sm shadow-sm focus:border-blue-500 focus:ring-2 focus:ring-blue-500 sm:p-2.5"
		  placeholder="Type your message..."
		></textarea>
	  </div>
  
	  {#if state.showSchedule}
		<ScheduleDatePicker
		  scheduleDate={state.scheduleDate}
		  scheduleTime={state.scheduleTime}
		  timezone={state.timezone}
		/>
	  {/if}
  
	  <div class="flex flex-col gap-3 sm:flex-row sm:justify-between">
		<div>
		  <Dialog.Close class="cursor-pointer rounded-lg bg-gray-400 px-4 py-2 text-white">Cancel</Dialog.Close>
		</div>
		<div class="flex flex-col gap-3 sm:flex-row">
		  <button
			type="button"
			class="flex w-full cursor-pointer items-center justify-center gap-2 rounded-lg bg-primary py-2 text-white sm:w-32"
			onclick={toggleSchedule}
		  >
			<i class="fa-regular fa-clock" ></i>
			Schedule
		  </button>
		  <button
			type="button"
			class="w-full cursor-pointer rounded-lg bg-blue-600 py-2 text-white sm:w-32"
			onclick={sendBlast}
		  >
			{state.showSchedule ? 'Schedule Blast' : 'Send Now'}
		  </button>
		</div>
	  </div>
	</form>
  </div>