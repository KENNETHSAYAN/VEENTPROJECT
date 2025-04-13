<script lang="ts">
    import Button from '$lib/components/ui/Button.svelte';
    import PaymentToggle from '$lib/components/ui/PaymentToggle.svelte';
    
    let ticketName = $state('');
    let validFrom = $state('');
    let validTo = $state('');
    let price = $state('');
    let quantity = $state('');
    let selectedColor = $state('#0FBA81');
    let isActivePayment = $state(false);
  
    const colors = ['#0066FF', '#F7D002', '#0FBA81', '#4B7B3B', '#DF4D60'];
  
    function handleAddTicket() {
      console.log({
        ticketName,
        validFrom,
        validTo,
        price,
        quantity,
        selectedColor,
        isActivePayment
      });
    }
    
    // In Svelte 5, we can directly set the state variable
    function handleTogglePayment(value: boolean) {
      isActivePayment = value;
    }
  
  </script>
  
  <div class="max-w-xl mx-auto p-8">
    <div class="space-y-6">
      <div class="grid grid-cols-2 gap-4">
        <div>
          <label for="ticket" class="block text-sm mb-2">Ticket Name</label>
          <input 
            type="text"
            bind:value={ticketName}
            placeholder="Enter ticket name"
            class="w-full p-3 bg-[#F8F9FC] rounded-md border-none"
          />
        </div>
        <div>
          <label for="validfrom" class="block text-sm mb-2">Valid from (DD/MM/YYYY)</label>
          <input 
            type="date"
            bind:value={validFrom}
            class="w-full p-3 bg-[#F8F9FC] rounded-md border-none"
          />
        </div>
      </div>
  
      <div class="grid grid-cols-2 gap-4">
        <div>
          <label for="price" class="block text-sm mb-2">Price</label>
          <input 
            type="number"
            bind:value={price}
            placeholder="Enter ticket price"
            class="w-full p-3 bg-[#F8F9FC] rounded-md border-none"
          />
        </div>
        <div>
          <label for="valid-in" class="block text-sm mb-2">Valid in (DD/MM/YYYY)</label>
          <input 
            type="date"
            bind:value={validTo}
            class="w-full p-3 bg-[#F8F9FC] rounded-md border-none"
          />
        </div>
      </div>
  
      <div class="grid grid-cols-2 gap-4">
        <div>
          <label for="quantity" class="block text-sm mb-2">Quantity</label>
          <input 
            type="number"
            bind:value={quantity}
            placeholder="Enter quantity"
            class="w-full p-3 bg-[#F8F9FC] rounded-md border-none"
          />
        </div>
        <div>
          <label for="activePayment" class="block text-sm mb-2">Active payment</label>
          <PaymentToggle 
            value={isActivePayment} 
            OnChange={handleTogglePayment}
          />
        </div>
      </div>
  
      <div>
        <label for="Label-color" class="block text-sm mb-2">Label Color</label>
        <div 
          class="p-3 rounded-md text-white mb-3"
          style="background-color: {selectedColor}"
        >
          {selectedColor}
        </div>
        <div class="flex gap-2">
          {#each colors as color}
            <button
              class="w-8 h-8 rounded-full border-2 transition-all"
              style="background-color: {color}; border-color: {selectedColor === color ? 'black' : 'transparent'}"
              onclick={() => selectedColor = color}
              aria-label="Select color {color}"
            ></button>
          {/each}
          <button
            class="w-8 h-8 rounded-full border-2 border-gray-300 flex items-center justify-center"
          >
            +
          </button>
        </div>
      </div>
  
      <div class="grid grid-cols-2 gap-4 pt-4">
        <Button
          onClick={handleAddTicket}
          label="Add Ticket"
          className="bg-[#DF4D60] text-white p-2 rounded-md"
        />
        <Button
          onClick={() => {}}
          label="Cancel"
          className="border border-gray-300 text-gray-700 p-2 rounded-md"
        />
      </div>
    </div>
  </div>