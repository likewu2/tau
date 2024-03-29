project(
  'tau', 'rust',
  version: '0.12.0',
  license: 'MIT',
)

dependency('glib-2.0', version: '>= 2.38')
dependency('gio-2.0', version: '>= 2.38')
dependency('cairo', version: '>= 1')
dependency('pango', version: '>=1.38')
dependency('gtk+-3.0', version: '>= 3.24')
dependency('libhandy-0.0', version: '>=0.0.10')

tau_prefix = get_option('prefix')
tau_bindir = join_paths(tau_prefix, get_option('bindir'))
tau_datadir = get_option('datadir')
tau_localedir = join_paths(tau_prefix, get_option('localedir'))
tau_libexecdir = join_paths(tau_prefix, get_option('libexecdir'), 'tau')
tau_plugin_dir = join_paths(tau_libexecdir, 'plugins')
tau_podir = join_paths(meson.current_source_dir(), 'po')

i18n = import('i18n')

cargo = find_program('cargo', required: true)
gresource = find_program('glib-compile-resources', required: true)
cargo_script = find_program('scripts/cargo.sh', required: true)

if get_option('profile') == 'development'
  profile = 'Devel'
  name_suffix = ' (Development)'
  vcs_tag = run_command('git', 'rev-parse', '--short', 'HEAD').stdout().strip()
  if vcs_tag == ''
    version_suffix = '-devel'
  else
    version_suffix = '-@0@'.format (vcs_tag)
  endif
else
  profile = ''
  name_suffix = ''
  version_suffix = ''
endif

application_id = 'org.gnome.Tau@0@'.format(profile)
name = 'Tau @0@'.format(name_suffix)
version = '@0@@1@'.format(meson.project_version(), version_suffix)

conf = configuration_data()

conf.set('application_id', application_id)

sources = files(
  'src/editview/src/main_state.rs',
  'src/editview/src/edit_view.rs',
  'src/editview/src/lib.rs',
  'src/editview/src/view_item.rs',
  'src/editview/src/fonts.rs',
  'src/editview/src/theme.rs',
  'src/editview/src/draw_invisible.rs',
  'src/editview/src/ui/close_tab.glade',
  'src/editview/src/ui/context_menu.glade',
  'src/editview/src/ui/find_replace.glade',
  'src/editview/src/ui/ev.glade',
  'src/editview/src/ui/resources.xml',
  'src/tau/src/main_win.rs',
  'src/tau/src/main.rs',
  'src/tau/src/errors.rs',
  'src/tau/src/about_win.rs',
  'src/tau/src/prefs_win.rs',
  'src/tau/src/globals.rs',
  'src/tau/src/syntax_config.rs',
  'src/tau/src/shortcuts_win.rs',
  'src/tau/src/ui/shortcuts_win.glade',
  'src/tau/src/ui/tau.glade',
  'src/tau/src/ui/prefs_win_handy.glade',
  'src/tau/src/ui/tau.glade',
  'src/tau/src/ui/app.css',
  'src/tau/src/ui/resources.xml',
  'src/tau-rpc/Cargo.toml',
  'src/tau-rpc/src/lib.rs',
  'src/tau-rpc/src/message.rs',
  'src/tau-rpc/src/errors.rs',
  'src/tau-rpc/src/client.rs',
  'src/tau-rpc/src/structs/alert.rs',
  'src/tau-rpc/src/structs/config.rs',
  'src/tau-rpc/src/structs/findreplace.rs',
  'src/tau-rpc/src/structs/language.rs',
  'src/tau-rpc/src/structs/line.rs',
  'src/tau-rpc/src/structs/mod.rs',
  'src/tau-rpc/src/structs/modifyselection.rs',
  'src/tau-rpc/src/structs/operation.rs',
  'src/tau-rpc/src/structs/plugins.rs',
  'src/tau-rpc/src/structs/position.rs',
  'src/tau-rpc/src/structs/scroll_to.rs',
  'src/tau-rpc/src/structs/style.rs',
  'src/tau-rpc/src/structs/theme.rs',
  'src/tau-rpc/src/structs/update.rs',
  'src/tau-rpc/src/structs/view.rs',
  'Cargo.toml',
  'Cargo.lock',
)

git = find_program('git', native: true, required: false)

subdir_xi_check = run_command('sh', '-c', 'ls -d $MESON_SOURCE_ROOT/vendor/xi-editor')
subdir_syntect_check = run_command('sh', '-c', 'ls -d $MESON_SOURCE_ROOT/vendor/xi-editor/rust/syntect-plugin/assets/')

if subdir_xi_check.returncode() != 0 or subdir_syntect_check.returncode() != 0
    if git.found()
        message('Couldn\'t find xi-editor subdir, trying to clone it')
        if run_command(git, 'status').returncode() == 128
            error('Can\'t clone submodule when not in a git repo! Make sure you\'re using the github release and not the automatically generated tarball!')
        endif
        submodule_update = run_command(git, 'submodule', 'update', '--init')
        message(submodule_update.stdout())
    else
        error('Couldn\'t find xi-editor subdir! If you\'re on a git checkout please install git and try again, if not make sure you\'re using the github release and not the automatically generated tarball!')
    endif
endif

xi_vendor_target_dir = join_paths(meson.current_build_dir(), 'vendor', 'xi-editor', 'rust')

syntect_build_path = join_paths(meson.current_source_dir(), 'vendor', 'xi-editor', 'rust', 'syntect-plugin')
syntect_dir = join_paths(tau_plugin_dir, 'syntect')
syntect_bindir = join_paths(syntect_dir, 'bin')

syntect_plugin_files = files(
  'vendor/xi-editor/rust/syntect-plugin/src/main.rs',
  'vendor/xi-editor/rust/syntect-plugin/src/stackmap.rs',
  'vendor/xi-editor/rust/syntect-plugin/Cargo.toml',
  'vendor/xi-editor/rust/Cargo.lock',
)

xi_syntect_plugin_dep = custom_target('cargo-build-syntect',
                        input: syntect_plugin_files,
                        output: ['xi-syntect-plugin'],
                        install: true,
                        install_dir: syntect_bindir,
                        install_mode: 'rwxr-xr-x',
                        console: true,
                        command: [cargo_script,
                                  syntect_build_path,
                                  join_paths(meson.build_root(),'@OUTDIR@'),
                                  '@OUTPUT@',
                                  xi_vendor_target_dir,
                                  'xi-syntect-plugin',
                                  tau_plugin_dir,
                                  tau_localedir,
                                  meson.project_version(),
                                  '',
                                  '',
                                  '',
                                  get_option('profile'),
                                 ])

synctect_assets = ['assets/Makefile.toml', 'assets/YAML.toml', 'manifest.toml']

foreach a: synctect_assets
    file = join_paths(syntect_build_path, a)
    install_data(
        file,
        install_dir: syntect_dir,
    )
endforeach

tau_depends = [xi_syntect_plugin_dep]

tau = custom_target('cargo-build-tau',
                        input: sources,
                        output: ['tau'],
                        install: true,
                        install_dir: tau_bindir,
                        install_mode: 'rwxr-xr-x',
                        console: true,
                        depends: tau_depends,
                        command: [cargo_script,
                                  meson.current_source_dir(),
                                  join_paths(meson.build_root(),'@OUTDIR@'),
                                  '@OUTPUT@',
                                  meson.current_build_dir(),
                                  'tau',
                                  tau_plugin_dir,
                                  tau_localedir,
                                  version,
                                  gresource.path(),
                                  application_id,
                                  name,
                                  get_option('profile'),
                                 ])

meson.add_install_script('meson_post_install.sh')

run_target(
  'release',
  command: ['scripts/release.sh', meson.project_name(), meson.project_version()]
)

run_target(
  'run',
  command: 'scripts/run.sh',
  depends: tau,
)

subdir('data')
subdir('po')

test(
  'cargo test --all',
  cargo,
  args: 'test',
  timeout: 3000,
)
