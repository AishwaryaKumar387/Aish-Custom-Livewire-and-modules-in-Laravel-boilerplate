# Aish-Custom-Livewire-and-modules-in-Laravel-boilerplate.
This is description of my code that acts as a plugin in Laravel boilerplate in which I created modules and custom livewire. You have to just follow the steps and modules will created.
To create modules, there is readme.md file inside.

STEPS FOR CUSTOM LIVEWIRE:

1. In <MODULE FOLDER>, inside "Http" folder, create a folder "Livewire", and inside that create a php file like <ClassName.php>
  example - ChannelsTable.php
  Code inside it will be like: (Channel as module)
  
  <?php

  namespace Modules\Channel\Http\Livewire;
  use Modules\Channel\Entities\Channel;
  use Livewire\WithPagination;
  use Livewire\Component;

  class ChannelsTable extends Component
  {

      public function render()
      {
          $channels = Channel::paginate(25);
          return view('channel::livewire.table')->with(compact('channels'));
      }
  }

2. Create view:
  Now inside view folder just create livewir folder and create table.balde file

  <div>
    <div class="table-responsive">
        <table class="table table-striped" >
            <thead>
                <tr>
                    <th scope="col">ID</th>
                    <th scope="col">Name</th>
                    <th scope="col">Created On</th>
                    <th scope="col">Updated On</th>
					<th scope="col">Actions</th>
                </tr>
            </thead>
         </table>
    </div>
</div>

3. Register livewire:
   In Providers folder just create a Custom provider "LivewireServiceProvider". (copy it from channel module)

  <?php 
    namespace Modules\<Module>\Providers;

    use Illuminate\Support\ServiceProvider;
    use Illuminate\Filesystem\Filesystem;
    use Nwidart\Modules\Facades\Module;
    use Illuminate\Support\Str;
    use Livewire\Livewire;

    class LivewireServiceProvider extends ServiceProvider
    {
        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            $this->loadComponents();
        }

        /**
         * Get the services provided by the provider.
         *
         * @return array
         */
        public function provides()
        {
            return [];
        }

        protected function loadComponents()
        {
            $modules = Module::toCollection();

            $filesystem = new Filesystem();

            $modules->map(function ($module) use ($filesystem) {
                $modulePath = $module->getPath();

                $moduleName = $module->getName();


                $path = $modulePath . '/Http/Livewire';

                $files = collect($filesystem->isDirectory($path) ? $filesystem->allFiles($path) : []);

                $files->map(function ($file) use ($moduleName, $path) {
                    $componentPath = Str::after($file->getPathname(), $path . '/');

                    $componentClassPath = strtr($componentPath, ['/' => '\\', '.php' => '']);

                    $componentName = $this->getComponentName($componentClassPath, $moduleName);

                    $componentClassStr = "\\Modules\\{$moduleName}\\Http\\Livewire\\" . $componentClassPath;

                    $componentClass = get_class(new $componentClassStr);

                    $loadComponent = Livewire::component($componentName, $componentClass);

                });
            });
        }

        protected function getComponentName($componentClassPath, $moduleName = null)
        {
            $dirs = explode('\\', $componentClassPath);

            $componentName = '';

            foreach ($dirs as $dir) {
                $componentName .= Str::kebab(lcfirst($dir)) . '.';
            }

            $moduleNamePrefix = ($moduleName) ? Str::lower($moduleName) . '::' : null;

            return Str::substr($moduleNamePrefix . $componentName, 0, -1);
        }
    }

   Then, in module, inside provider folder there will be a file <ModuleNameServiceProvider">. Open it:
    In that there will be a function "register", just add:
    $this->app->register(LivewireServiceProvider::class);
   Note: LivewireServiceProvider should be class name of custom livewire provider.

4. Use:
  where you want to use:
   just write like <livewire:MODULE_NAME::LivewireClass/>
   eg. <livewire:channel::channels-table/>


