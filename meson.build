project(
  'Hyprland',
  'cpp',
  'c',
  version: run_command('cat', join_paths(meson.project_source_root(), 'VERSION'), check: true).stdout().strip(),
  default_options: [
    'warning_level=2',
    'default_library=static',
    'optimization=3',
    'buildtype=release',
    'debug=false',
    'cpp_std=c++26',
  ],
)

datarootdir = '-DDATAROOTDIR="' + get_option('prefix') / get_option('datadir') + '"'
add_project_arguments(
  [
    '-Wno-unused-parameter',
    '-Wno-unused-value',
    '-Wno-missing-field-initializers',
    '-Wno-narrowing',
    '-Wno-pointer-arith', datarootdir,
  ],
  language: 'cpp',
)

cpp_compiler = meson.get_compiler('cpp')
if cpp_compiler.check_header('execinfo.h')
  add_project_arguments('-DHAS_EXECINFO', language: 'cpp')
endif

xcb_dep = dependency('xcb', required: get_option('xwayland'))
xcb_composite_dep = dependency('xcb-composite', required: get_option('xwayland'))
xcb_errors_dep = dependency('xcb-errors', required: get_option('xwayland'))
xcb_icccm_dep = dependency('xcb-icccm', required: get_option('xwayland'))
xcb_render_dep = dependency('xcb-render', required: get_option('xwayland'))
xcb_res_dep = dependency('xcb-res', required: get_option('xwayland'))
xcb_xfixes_dep = dependency('xcb-xfixes', required: get_option('xwayland'))

gio_dep = dependency('gio-2.0', required: true)

cmake = import('cmake')
udis = cmake.subproject('udis86')
udis86 = udis.dependency('libudis86')

if not xcb_dep.found()
  add_project_arguments('-DNO_XWAYLAND', language: 'cpp')
endif

backtrace_dep = cpp_compiler.find_library('execinfo', required: false)
epoll_dep = dependency('epoll-shim', required: false) # timerfd on BSDs

# Handle options
if get_option('systemd').enabled()
  add_project_arguments('-DUSES_SYSTEMD', language: 'cpp')
endif

if get_option('legacy_renderer').enabled()
  add_project_arguments('-DLEGACY_RENDERER', language: 'cpp')
endif

if get_option('buildtype') == 'debug'
  add_project_arguments('-DHYPRLAND_DEBUG', language: 'cpp')
endif

# Generate hyprland version and populate version.h
run_command('sh', '-c', 'scripts/generateVersion.sh', check: true)

# Install headers
globber = run_command('find', 'src', '-name', '*.h*', check: true)
headers = globber.stdout().strip().split('\n')
foreach file : headers
  install_headers(file, subdir: 'hyprland', preserve_path: true)
endforeach

subdir('protocols')
subdir('src')
subdir('hyprctl')
subdir('hyprpm/src')
subdir('assets')
subdir('example')
subdir('docs')

# Generate hyprland.pc
pkg_install_dir = join_paths(get_option('datadir'), 'pkgconfig')

import('pkgconfig').generate(
  name: 'Hyprland',
  filebase: 'hyprland',
  url: 'https://github.com/hyprwm/Hyprland',
  description: 'Hyprland header files',
  install_dir: pkg_install_dir,
  subdirs: ['', 'hyprland/protocols', 'hyprland'],
)
