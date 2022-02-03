<h1 class="code-line" data-line-start=0 data-line-end=1 ><a id="AishCustomLivewireandmodulesinLaravelboilerplate_0"></a>Aish-Custom-Livewire-and-modules-in-Laravel-boilerplate.</h1>
<p class="has-line-data" data-line-start="1" data-line-end="3">This is description of my code that acts as a plugin in Laravel boilerplate in which I created modules and custom livewire. You have to just follow the steps and modules will created.<br>
To create modules, there is <a href="http://readme.md">readme.md</a> file inside.</p>
<p class="has-line-data" data-line-start="4" data-line-end="5">STEPS FOR CUSTOM LIVEWIRE:</p>
<ol>
<li class="has-line-data" data-line-start="6" data-line-end="10">In &lt;MODULE FOLDER&gt;, inside “Http” folder, create a folder “Livewire”, and inside that create a php file like &lt;ClassName.php&gt;<br>
example - ChannelsTable.php<br>
Code inside it will be like: (Channel as module)</li>
</ol>
<p class="has-line-data" data-line-start="10" data-line-end="11">&lt;?php</p>
<p class="has-line-data" data-line-start="12" data-line-end="16">namespace Modules\Channel\Http\Livewire;<br>
use Modules\Channel\Entities\Channel;<br>
use Livewire\WithPagination;<br>
use Livewire\Component;</p>
<p class="has-line-data" data-line-start="17" data-line-end="19">class ChannelsTable extends Component<br>
{</p>
<pre><code>  public function render()
  {
      $channels = Channel::paginate(25);
      return view('channel::livewire.table')-&gt;with(compact('channels'));
  }
</code></pre>
<p class="has-line-data" data-line-start="25" data-line-end="26">}</p>
<ol start="2">
<li class="has-line-data" data-line-start="28" data-line-end="30">Create view:<br>
Now inside view folder just create livewir folder and create table.balde file</li>
</ol>
<p class="has-line-data" data-line-start="32" data-line-end="47">&lt;div&gt;<br>
&lt;div class=“table-responsive”&gt;<br>
&lt;table class=“table table-striped” &gt;<br>
&lt;thead&gt;<br>
&lt;tr&gt;<br>
&lt;th scope=“col”&gt;ID&lt;/th&gt;<br>
&lt;th scope=“col”&gt;Name&lt;/th&gt;<br>
&lt;th scope=“col”&gt;Created On&lt;/th&gt;<br>
&lt;th scope=“col”&gt;Updated On&lt;/th&gt;<br>
&lt;th scope=“col”&gt;Actions&lt;/th&gt;<br>
&lt;/tr&gt;<br>
&lt;/thead&gt;<br>
&lt;/table&gt;<br>
&lt;/div&gt;<br>
&lt;/div&gt;</p>
<ol start="3">
<li class="has-line-data" data-line-start="49" data-line-end="52">Register livewire:<br>
In Providers folder just create a Custom provider “LivewireServiceProvider”. (copy it from channel module)</li>
</ol>
<p class="has-line-data" data-line-start="52" data-line-end="54">&lt;?php<br>
namespace Modules&lt;Module&gt;\Providers;</p>
<pre><code>use Illuminate\Support\ServiceProvider;
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
        $this-&gt;loadComponents();
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

        $modules-&gt;map(function ($module) use ($filesystem) {
            $modulePath = $module-&gt;getPath();

            $moduleName = $module-&gt;getName();


            $path = $modulePath . '/Http/Livewire';

            $files = collect($filesystem-&gt;isDirectory($path) ? $filesystem-&gt;allFiles($path) : []);

            $files-&gt;map(function ($file) use ($moduleName, $path) {
                $componentPath = Str::after($file-&gt;getPathname(), $path . '/');

                $componentClassPath = strtr($componentPath, ['/' =&gt; '\\', '.php' =&gt; '']);

                $componentName = $this-&gt;getComponentName($componentClassPath, $moduleName);

                $componentClassStr = &quot;\\Modules\\{$moduleName}\\Http\\Livewire\\&quot; . $componentClassPath;

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
</code></pre>
<p class="has-line-data" data-line-start="132" data-line-end="136">Then, in module, inside provider folder there will be a file &lt;ModuleNameServiceProvider&quot;&gt;. Open it:<br>
In that there will be a function “register”, just add:<br>
$this-&gt;app-&gt;register(LivewireServiceProvider::class);<br>
Note: LivewireServiceProvider should be class name of custom livewire provider.</p>
<ol start="4">
<li class="has-line-data" data-line-start="137" data-line-end="141">Use:<br>
where you want to use:<br>
just write like &lt;livewire:MODULE_NAME::LivewireClass/&gt;<br>
eg. &lt;livewire:channel::channels-table/&gt;</li>
</ol>
