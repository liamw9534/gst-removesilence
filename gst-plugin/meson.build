plugin_c_args = ['-DHAVE_CONFIG_H']

cdata = configuration_data()
cdata.set_quoted('PACKAGE_VERSION', gst_version)
cdata.set_quoted('PACKAGE', 'gst-removesilence')
cdata.set_quoted('GST_LICENSE', 'LGPL')
cdata.set_quoted('GST_API_VERSION', api_version)
cdata.set_quoted('GST_PACKAGE_NAME', 'Remove silence gst plugin')
cdata.set_quoted('GST_PACKAGE_ORIGIN', 'https://github.com/liamw9534')
configure_file(output : 'config.h', configuration : cdata)

gst_dep = dependency('gstreamer-1.0',
  fallback : ['gstreamer', 'gst_dep'])
gstbase_dep = dependency('gstreamer-base-1.0',
  fallback : ['gstreamer', 'gst_base_dep'])
gstaudio_dep = dependency('gstreamer-audio-1.0',
    fallback : ['gst-plugins-base', 'audio_dep'])
libm = cc.find_library('m', required : false)

sources = [
  'src/gstremovesilence.c', 'src/vad_private.c'
  ]

gstremovesilence = library('gstremovesilence',
  sources,
  c_args: plugin_c_args,
  dependencies : [gst_dep, gstaudio_dep, libm],
  install : true,
  install_dir : plugins_install_dir,
)
