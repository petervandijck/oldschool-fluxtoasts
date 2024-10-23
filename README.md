# Using Flux Toasts in oldschool controllers

Flux toast messages are really nice, but they are built to be used only with Livewire. 
I use a lot of traditional non-Livewire Controllers with pageviews, and I wanted to use them there as well.
After a good chat with Claude.ai, we created this approach which works really well.
- See: https://fluxui.dev/components/toast
## Installation
1. Create the Livewire component:
```bash
php artisan make:livewire ToastHandler
```
2. Replace `app/Livewire/ToastHandler.php` content with:
```php
<?php

namespace App\Livewire;

use Livewire\Component;
use Flux;

class ToastHandler extends Component
{
    public $hasToast = false;
    public $toastData = [];

    public function mount()
    {
        if (session()->has('toast')) {
            $this->hasToast = true;
            $this->toastData = session('toast');
            session()->forget('toast');
        }
    }

    public function showToast($data)
    {
        $data = json_decode($data, true);
        
        $params = array_filter([
            'text' => $data['message'] ?? null,
            'heading' => $data['heading'] ?? null,
            'variant' => $data['variant'] ?? null,
            'duration' => $data['duration'] ?? null,
            'position' => $data['position'] ?? null,
        ], fn($value) => !is_null($value));

        Flux::toast(...$params);
    }

    public function render()
    {
        return <<<'blade'
            <div>
                @if($hasToast)
                    <div wire:init="showToast('{{ json_encode($this->toastData) }}')"></div>
                @endif
            </div>
        blade;
    }
}
```
The render() function will use wire:init to kick off the toast, using the data that we sent via the session flash message.

3. Create and add content to `resources/views/livewire/toast-handler.blade.php`:
This is weird, but we need an empty Livewire component to act as a "listener". It can stay empty.
```blade
<div>
    {{-- Empty div required for Livewire component --}}
</div>
```

4. Add to your layout (e.g. `resources/views/layouts/app.blade.php`):
```blade
<flux:toast position="top right" />
<livewire:toast-handler />
```

## Usage

Now in your Controllers, you can do something like this:

```php
session()->flash('toast', [
    'message' => 'Changes saved successfully',
    'variant' => 'success',
    'heading' => 'Saved',    // optional
    'duration' => 3000,      // optional, in milliseconds
    'position' => 'top right' // optional
]);

return redirect()->back();
```

## Available Options
- `message`: Toast message text
- `variant`: 'success', 'warning', or 'danger'
- `heading`: Optional header text
- `duration`: Time in milliseconds (default 5000, 0 for permanent)
- `position`: Toast position (e.g. 'top right', 'bottom left')