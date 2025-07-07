project('resize2', 'cpp',
  default_options : ['buildtype=release', 'b_ndebug=if-release', 'c_std=c89', 'cpp_std=c++17'],
  meson_version : '>=1.0.0',
  version : '0.3.3'
)

sources = [
    'resize.cpp'
]

deps = []
vs_plugin_dir = ''

if host_machine.system() == 'windows'
  # Path to zimg/_msvc/x64/Release, compile zimg first
  zimg_dir = get_option('zimg_dir')
  # The VapourSynth/sdk directory containing the include and lib64 folder
  vs_dir = get_option('vs_dir')

  zimg_include_dir = join_paths(zimg_dir, '..', '..', '..', 'src')
  zimg_zimg = join_paths(zimg_dir, '..', '..', '..', 'src', 'zimg')
  zimg_lib = join_paths(zimg_dir, 'z.lib')
  graphengine_include_dir = join_paths(zimg_dir, '..', '..', '..', 'graphengine', 'include')

  zimg_dep = declare_dependency(
    include_directories: [
      include_directories(zimg_include_dir),
      include_directories(zimg_zimg),
      include_directories(graphengine_include_dir)
    ],
    link_args: [zimg_lib]
  )

  vs_include_dir = join_paths(vs_dir, 'include', 'vapoursynth')
  vs_lib = join_paths(vs_dir, 'lib64', 'vapoursynth.lib')

  vapoursynth_dep = declare_dependency(
    include_directories: include_directories(vs_include_dir),
    link_args: [vs_lib]
  )

  deps = [vapoursynth_dep, zimg_dep]
  
  # This was a mistake
  appdata_res = run_command('cmd', '/C', 'echo %APPDATA%', check: true)
  appdata = appdata_res.stdout().strip()
  if appdata == '' or appdata == '%APPDATA%'
    error('Cannot detect %APPDATA%, erm what the sigma')
  endif
  vs_plugin_dir = join_paths(appdata, 'VapourSynth', 'plugins64')

else
  vapoursynth_dep = dependency('vapoursynth').partial_dependency(compile_args : true, includes : true)

  deps = [
      vapoursynth_dep,
      dependency('zimg_patched'),
  ]

  vs_plugin_dir = join_paths(vapoursynth_dep.get_variable(pkgconfig: 'libdir'), 'vapoursynth')

endif

shared_module('resize2', sources,
    dependencies: deps,
    link_args: host_machine.system() == 'windows' ? [] : ['-static'],
    install: true,
    install_dir: vs_plugin_dir,
    gnu_symbol_visibility: 'hidden',
)