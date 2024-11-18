<script lang="ts">
  import { writable, type Writable } from 'svelte/store';
  import { Button } from "$lib/components/ui/button";
  import { Input } from "$lib/components/ui/input";
  import { Card } from "$lib/components/ui/card";
  import { Checkbox } from "$lib/components/ui/checkbox";
  import { Switch } from "$lib/components/ui/switch";
  import { Gamepad, Settings, Gamepad2 } from 'lucide-svelte';

  type BillingCycle = 'monthly' | 'yearly';
  type PlanType = 'arcade' | 'advanced' | 'pro';
  type AddonType = 'onlineService' | 'largerStorage' | 'customProfile';

  interface FormData {
    name: string;
    email: string;
    phone: string;
    plan: PlanType;
    addons: Record<AddonType, boolean>;
  }

  interface FormErrors {
    name: string;
    email: string;
    phone: string;
  }

  interface PlanPricing {
    monthly: Record<PlanType, number>;
    yearly: Record<PlanType, number>;
  }

  interface AddonPricing {
    monthly: Record<AddonType, number>;
    yearly: Record<AddonType, number>;
  }

  const currentStep = writable(1);
  const billingCycle = writable<BillingCycle>('monthly');
  const formData = writable<FormData>({
    name: '',
    email: '',
    phone: '',
    plan: 'arcade',
    addons: {
      onlineService: false,
      largerStorage: false,
      customProfile: false
    }
  });
  const formErrors = writable<FormErrors>({
    name: '',
    email: '',
    phone: ''
  });
  const isSubmitting = writable(false);
  let hoveredPlan: PlanType | null = null;

  const plans: PlanPricing = {
    monthly: {
      arcade: 9,
      advanced: 12,
      pro: 15
    },
    yearly: {
      arcade: 90,
      advanced: 120,
      pro: 150
    }
  };

  const addons: AddonPricing = {
    monthly: {
      onlineService: 1,
      largerStorage: 2,
      customProfile: 2
    },
    yearly: {
      onlineService: 10,
      largerStorage: 20,
      customProfile: 20
    }
  };

  function validateEmail(email: string): boolean {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  }

  function validatePhone(phone: string): boolean {
    return /^\+?[\d\s-]{10,}$/.test(phone);
  }

  function validateStep1(): boolean {
    let isValid = true;
    $formErrors = {
      name: '',
      email: '',
      phone: ''
    };

    if (!$formData.name) {
      $formErrors.name = 'This field is required';
      isValid = false;
    }

    if (!$formData.email) {
      $formErrors.email = 'This field is required';
      isValid = false;
    } else if (!validateEmail($formData.email)) {
      $formErrors.email = 'Invalid email format';
      isValid = false;
    }

    if (!$formData.phone) {
      $formErrors.phone = 'This field is required';
      isValid = false;
    } else if (!validatePhone($formData.phone)) {
      $formErrors.phone = 'Invalid phone format';
      isValid = false;
    }

    return isValid;
  }

  async function nextStep() {
    if ($currentStep === 1 && !validateStep1()) {
      return;
    }

    if ($currentStep === 4) {
      $isSubmitting = true;
      try {
        // Simulate API call
        await new Promise(resolve => setTimeout(resolve, 1000));
        $currentStep = 5;
      } catch (error) {
        console.error('Submission error:', error);
      } finally {
        $isSubmitting = false;
      }
    } else {
      $currentStep = Math.min($currentStep + 1, 5);
    }
  }

  function prevStep() {
    $currentStep = Math.max($currentStep - 1, 1);
  }

  function toggleBilling() {
    $billingCycle = $billingCycle === 'monthly' ? 'yearly' : 'monthly';
  }

  function handlePlanChange(plan: PlanType) {
    $formData.plan = plan;
  }

  function handleAddonChange(addon: AddonType) {
    $formData.addons[addon] = !$formData.addons[addon];
  }

  function calculateTotal(): number {
    let total = plans[$billingCycle][$formData.plan];
    Object.entries($formData.addons).forEach(([key, value]) => {
      if (value) {
        total += addons[$billingCycle][key as AddonType];
      }
    });
    return total;
  }

  function handleInputChange(field: keyof FormErrors) {
    $formErrors[field] = '';
  }

  const plansList: PlanType[] = ['arcade', 'advanced', 'pro'];
  const addonsList: AddonType[] = ['onlineService', 'largerStorage', 'customProfile'];
</script>

<div class="flex h-full min-h-screen items-center justify-center bg-slate-50 p-0 md:p-4">
  <Card class="flex h-screen w-full max-w-[940px] lg:max-w-max flex-col gap-0 border-none p-0 shadow-none md:h-auto md:flex-row md:gap-4 md:border md:p-4 md:shadow">
    <!-- Sidebar -->
    <div class="relative min-h-[172px] w-full overflow-hidden rounded-none bg-[#6259FF] p-8 text-white md:min-h-0 md:w-[274px] md:rounded-lg">
      <div class="relative z-10 flex justify-center gap-4 md:flex-col md:gap-0">
        {#each [1, 2, 3, 4] as step}
          <div class="mb-8 flex items-center gap-4">
            <div class="flex h-8 w-8 items-center justify-center rounded-full border-2 {$currentStep === step ? 'border-[#BEE2FD] bg-[#BEE2FD] text-black' : ''}">
              {step}
            </div>
            <div class="hidden md:block">
              <div class="text-sm text-slate-300">STEP {step}</div>
              <div class="font-bold">
                {['YOUR INFO', 'SELECT PLAN', 'ADD-ONS', 'SUMMARY'][step - 1]}
              </div>
            </div>
          </div>
        {/each}
      </div>
      <!-- Decorative background -->
      <div class="absolute bottom-0 left-0 right-0">
        <div class="absolute bottom-[-80px] right-[-20px] h-40 w-40 rounded-full bg-[#F9818E] opacity-70"></div>
        <div class="absolute bottom-[-60px] left-[-20px] h-40 w-40 rounded-full bg-[#FFAF7E] opacity-70"></div>
      </div>
    </div>

    <!-- Content -->
    <div class="flex-1 p-4 shadow-none md:border-none md:p-6">
      <Card class="relative z-30 -mt-20 h-max p-5 pb-16 pt-10 md:-mt-0 md:border-none md:p-0 md:pb-0 md:pt-0 md:shadow-none">
        {#if $currentStep === 1}
          <div class="max-w-lg">
            <h1 class="mb-2 text-3xl font-bold text-[#02295A]">Personal info</h1>
            <p class="mb-8 text-gray-400">Please provide your name, email address, and phone number.</p>

            <div class="space-y-8">
              <div>
                <label for="name" class="text-sm text-[#02295A]">Name</label>
                <Input
                  type="text"
                  id="name"
                  placeholder="e.g. Stephen King"
                  bind:value={$formData.name}
                  class={$formErrors.name ? 'border-red-500' : ''}
                  aria-invalid={!!$formErrors.name}
                  aria-describedby={$formErrors.name ? 'name-error' : undefined}
                  on:input={() => handleInputChange('name')}
                />
                {#if $formErrors.name}
                  <p id="name-error" class="mt-1 text-sm text-red-500">{$formErrors.name}</p>
                {/if}
              </div>

              <div>
                <label for="email" class="text-sm text-[#02295A]">Email Address</label>
                <Input
                  type="email"
                  id="email"
                  placeholder="e.g. stephenking@lorem.com"
                  bind:value={$formData.email}
                  class={$formErrors.email ? 'border-red-500' : ''}
                  aria-invalid={!!$formErrors.email}
                  aria-describedby={$formErrors.email ? 'email-error' : undefined}
                  on:input={() => handleInputChange('email')}
                />
                {#if $formErrors.email}
                  <p id="email-error" class="mt-1 text-sm text-red-500">{$formErrors.email}</p>
                {/if}
              </div>

              <div>
                <label for="phone" class="text-sm text-[#02295A]">Phone Number</label>
                <Input
                  type="tel"
                  id="phone"
                  placeholder="e.g. +1 234 567 890"
                  bind:value={$formData.phone}
                  class={$formErrors.phone ? 'border-red-500' : ''}
                  aria-invalid={!!$formErrors.phone}
                  aria-describedby={$formErrors.phone ? 'phone-error' : undefined}
                  on:input={() => handleInputChange('phone')}
                />
                {#if $formErrors.phone}
                  <p id="phone-error" class="mt-1 text-sm text-red-500">{$formErrors.phone}</p>
                {/if}
              </div>
            </div>
          </div>
        {/if}

        {#if $currentStep === 2}
          <div>
            <h1 class="mb-2 text-3xl font-bold text-[#02295A]">Select your plan</h1>
            <p class="mb-8 text-gray-400">You have the option of monthly or yearly billing.</p>

            <div class="mb-8 flex flex-col gap-4 md:flex-row">
              {#each plansList as plan}
                <div
                  class="max-w-none flex-1 md:max-w-[138px]"
                  role="button"
                  tabindex="0"
                  on:mouseenter={() => hoveredPlan = plan}
                  on:mouseleave={() => hoveredPlan = null}
                  on:click={() => handlePlanChange(plan)}
                  on:keydown={(e) => {
                    if (e.key === 'Enter' || e.key === ' ') {
                      handlePlanChange(plan);
                    }
                  }}
                >
                  <label
                    class="flex md:block h-20 md:h-auto gap-4 md:gap-0 cursor-pointer rounded-lg border p-4 transition-all
                    {$formData.plan === plan ? 'border-[#6259FF] bg-slate-50' : 
                     hoveredPlan === plan ? 'border-[#6259FF]' : 'border-gray-200'}"
                  >
                    {#if plan === 'arcade'}
                      <Gamepad class="mb-10 h-10 w-10 text-[#FFAF7E]" />
                    {:else if plan === 'advanced'}
                      <Settings class="mb-10 h-10 w-10 text-[#F9818E]" />
                    {:else}
                      <Gamepad2 class="mb-10 h-10 w-10 text-[#6259FF]" />
                    {/if}
                    <div>
                      <div class="font-medium text-[#02295A] capitalize">{plan}</div>
                      <div class="text-sm text-gray-400">
                        ${plans[$billingCycle][plan]}/{$billingCycle === 'monthly' ? 'mo' : 'yr'}
                      </div>
                    </div>
                    {#if $billingCycle === 'yearly'}
                      <div class="text-sm text-[#02295A]">2 months free</div>
                    {/if}
                    <input
                      type="radio"
                      name="plan"
                      value={plan}
                      checked={$formData.plan === plan}
                      class="hidden"
                    />
                  </label>
                </div>
              {/each}
            </div>

            <div class="flex items-center justify-center gap-6 rounded-lg bg-slate-50 p-3">
              <span class="text-sm font-medium {$billingCycle === 'monthly' ? 'text-[#02295A]' : 'text-gray-400'}">
                Monthly
              </span>
              <Switch checked={$billingCycle === 'yearly'} onCheckedChange={toggleBilling} />
              <span class="text-sm font-medium {$billingCycle === 'yearly' ? 'text-[#02295A]' : 'text-gray-400'}">
                Yearly
              </span>
            </div>
          </div>
        {/if}

        {#if $currentStep === 3}
          <div>
            <h1 class="mb-2 text-3xl font-bold text-[#02295A]">Pick add-ons</h1>
            <p class="mb-8 text-gray-400">Add-ons help enhance your gaming experience.</p>

            <div class="space-y-4">
              {#each addonsList as addon}
                <label
                  class="flex cursor-pointer items-center rounded-lg border p-4 transition-all
                  {$formData.addons[addon] ? 'border-[#6259FF] bg-slate-50' : 'border-gray-200'}
                  hover:border-[#6259FF]"
                >
                  <Checkbox 
                    checked={$formData.addons[addon]} 
                    onCheckedChange={() => handleAddonChange(addon)}
                  />
                  <div class="ml-6 flex-1">
                    <div class="font-medium text-[#02295A]">
                      {addon === 'onlineService' ? 'Online service' :
                       addon === 'largerStorage' ? 'Larger storage' : 'Customizable profile'}
                    </div>
                    <div class="text-sm text-gray-400">
                      {addon === 'onlineService' ? 'Access to multiplayer games' :
                       addon === 'largerStorage' ? 'Extra 1TB of cloud save' : 'Custom theme on your profile'}
                    </div>
                  </div>
                  <div class="text-sm text-[#6259FF]">
                    +${addons[$billingCycle][addon]}/{$billingCycle === 'monthly' ? 'mo' : 'yr'}
                  </div>
                </label>
              {/each}
            </div>
          </div>
        {/if}

        {#if $currentStep === 4}
          <div>
            <h1 class="mb-2 text-3xl font-bold text-[#02295A]">Finishing up</h1>
            <p class="mb-8 text-gray-400">Double-check everything looks OK before confirming.</p>

            <div class="rounded-lg bg-slate-50 p-6">
              <div class="flex items-center justify-between border-b pb-6">
                <div>
                  <div class="font-medium capitalize text-[#02295A]">
                    {$formData.plan} ({$billingCycle})
                  </div>
                  <button
                    class="text-sm text-gray-400 underline"
                    on:click={() => $currentStep = 2}
                  >
                    Change
                  </button>
                </div>
                <div class="font-medium text-[#02295A]">
                  ${plans[$billingCycle][$formData.plan]}/{$billingCycle === 'monthly' ? 'mo' : 'yr'}
                </div>
              </div>

              {#if Object.values($formData.addons).some(Boolean)}
                <div class="space-y-4 pt-4">
                  {#each addonsList as addon}
                    {#if $formData.addons[addon]}
                      <div class="flex justify-between">
                        <div class="text-gray-400">
                          {addon === 'onlineService' ? 'Online service' :
                           addon === 'largerStorage' ? 'Larger storage' : 'Customizable profile'}
                        </div>
                        <div class="text-[#02295A]">
                          +${addons[$billingCycle][addon]}/{$billingCycle === 'monthly' ? 'mo' : 'yr'}
                        </div>
                      </div>
                    {/if}
                  {/each}
                </div>
              {/if}
            </div>

            <div class="flex items-center justify-between p-6">
              <div class="text-gray-400">
                Total (per {$billingCycle === 'monthly' ? 'month' : 'year'})
              </div>
              <div class="text-xl font-bold text-[#6259FF]">
                +${calculateTotal()}/{$billingCycle === 'monthly' ? 'mo' : 'yr'}
              </div>
            </div>
          </div>
        {/if}

        {#if $currentStep === 5}
          <div class="mx-auto flex h-full max-w-md flex-col items-center justify-center text-center">
            <div class="mb-8 flex h-20 w-20 items-center justify-center rounded-full bg-[#F9818E]">
              <svg
                xmlns="http://www.w3.org/2000/svg"
                width="40"
                height="40"
                viewBox="0 0 24 24"
                fill="none"
                stroke="white"
                stroke-width="2"
                stroke-linecap="round"
                stroke-linejoin="round"
              >
                <polyline points="20 6 9 17 4 12"></polyline>
              </svg>
            </div>
            <h1 class="mb-4 text-3xl font-bold text-[#02295A]">Thank you!</h1>
            <p class="text-gray-400">
              Thanks for confirming your subscription! We hope you have fun using our platform.
              If you ever need support, please feel free to email us at support@loremgaming.com.
            </p>
          </div>
        {/if}

        {#if $currentStep < 5}
          <div class="mt-auto hidden justify-between pt-16 md:flex">
            {#if $currentStep > 1}
              <Button variant="ghost" on:click={prevStep} disabled={$isSubmitting}>
                Go Back
              </Button>
            {:else}
              <div></div>
            {/if}
            <Button
              variant={$currentStep === 4 ? 'default' : 'secondary'}
              class={$currentStep === 4 ? 'bg-[#6259FF] hover:bg-[#928CFF]' : ''}
              on:click={nextStep}
              disabled={$isSubmitting}
            >
              {$isSubmitting
                ? 'Submitting...'
                : $currentStep === 4
                ? 'Confirm'
                : 'Next Step'}
            </Button>
          </div>
        {/if}
      </Card>

      {#if $currentStep < 5}
        <div class="me-6 mt-10 flex w-full justify-between mb-4 md:mb-6 md:mt-0 md:hidden">
          {#if $currentStep > 1}
            <Button variant="ghost" on:click={prevStep} disabled={$isSubmitting}>
              Go Back
            </Button>
          {:else}
            <div></div>
          {/if}
          <Button
            variant={$currentStep === 4 ? 'default' : 'secondary'}
            class={$currentStep === 4 ? 'bg-[#6259FF] hover:bg-[#928CFF]' : ''}
            on:click={nextStep}
            disabled={$isSubmitting}
          >
            {$isSubmitting
              ? 'Submitting...'
              : $currentStep === 4
              ? 'Confirm'
              : 'Next Step'}
          </Button>
        </div>
      {/if}
    </div>
  </Card>
</div>

<style>
  [data-state="checked"] {
    border-color: #6259FF;
    background-color: rgb(248 250 252);
  }

  [data-state="unchecked"] {
    border-color: #e5e7eb;
  }

  [data-state="checked"]:hover,
  [data-state="unchecked"]:hover {
    border-color: #6259FF;
  }
</style>