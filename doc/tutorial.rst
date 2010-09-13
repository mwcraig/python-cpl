Tutorial
========

Simple example
--------------

The following code takes BIAS input file names from the command line and writes the
MASTER BIAS to the file name provided with the :option:`-o` option::

  from optparse import OptionParser
  import sys
  import cpl

  parser = OptionParser(usage='%prog files')
  parser.add_option('-o', '--output', help='Output file', default='master_bias.fits')

  (opt, filenames) = parser.parse_args()
  if not filenames:
      parser.print_help()
      sys.exit()

  cpl.esorex.init()

  cpl.msg.level = 'info'  
  cpl.msg.logfile('muse_pipeline.log', level = 'debug')

  muse_bias = cpl.Recipe('muse_bias')
  muse_bias.nifu = 1

  res = muse_bias(filenames)
  res.MASTER_BIAS.writeto(opt.output)
  
Quick guide
-----------

Input lines are indicated with ">>>" (the python prompt).
The package can be imported with

>>> import cpl

This import statement will fail if the CPL libraries are not found.  
Init the search path for CPL recipes from the esorex startup and list
available recipes:

>>> cpl.esorex.init()
>>> cpl.Recipe.list()
[('muse_quick_image', ['0.2.0', '0.3.0']),
 ('muse_scipost', ['0.2.0', '0.3.0']),
 ('muse_scibasic', ['0.2.0', '0.3.0']),
 ('muse_flat', ['0.2.0', '0.3.0']),
 ('muse_subtract_sky', ['0.2.0', '0.3.0']),
 ('muse_bias', ['0.2.0', '0.3.0']),
 ('muse_ronbias', ['0.2.0', '0.3.0']),
 ('muse_fluxcal', ['0.2.0', '0.3.0']),
 ('muse_focus', ['0.2.0', '0.3.0']),
 ('muse_lingain', ['0.2.0', '0.3.0']),
 ('muse_dark', ['0.2.0', '0.3.0']),
 ('muse_combine_pixtables', ['0.2.0', '0.3.0']),
 ('muse_astrometry', ['0.2.0', '0.3.0']),
 ('muse_wavecal', ['0.2.0', '0.3.0']),
 ('muse_exp_combine', ['0.2.0', '0.3.0']),
 ('muse_dar_correct', ['0.2.0', '0.3.0']),
 ('muse_standard', ['0.2.0', '0.3.0']),
 ('muse_create_sky', ['0.2.0', '0.3.0']),
 ('muse_apply_astrometry', ['0.2.0', '0.3.0']),
 ('muse_rebin', ['0.2.0', '0.3.0'])]

To create a recipe specified by name:

>>> muse_scibasic = cpl.Recipe('muse_scibasic')

List all parameters:

>>> print muse_scibasic.param
[Parameter('nifu', default=99), Parameter('cr', default=none), 
 Parameter('xbox', default=15), Parameter('ybox', default=40), 
 Parameter('passes', default=2), Parameter('thres', default=4.5), 
 Parameter('resample', default=False), Parameter('dlambda', default=1.25)]

Set a parameter:

>>> muse_scibasic.param.nifu = 1

Print the value of a parameter (:keyword:`None` if the parameter is set to default)

>>> print muse_scibasic.param.nifu.value
1

List all calibration frames:

>>> print muse_scibasic.calib
[FrameDef('TRACE_TABLE', value=None), FrameDef('WAVECAL_TABLE', value=None), 
 FrameDef('MASTER_BIAS', value=None), FrameDef('MASTER_DARK', value=None), 
 FrameDef('GEOMETRY_TABLE', value=None), FrameDef('BADPIX_TABLE', value=None), 
 FrameDef('MASTER_FLAT', value=None)]

Set calibration frames with files:

>>> muse_scibasic.calib.MASTER_BIAS    = 'MASTER_BIAS-01.fits'
>>> muse_scibasic.calib.MASTER_FLAT    = 'MASTER_FLAT-01.fits'
>>> muse_scibasic.calib.TRACE_TABLE    = 'TRACE_TABLE-01.fits'
>>> muse_scibasic.calib.GEOMETRY_TABLE = 'geometry_table.fits'

Set calibration frame with :class:`pyfits.HDUList`:

>>> import pyfits
>>> wavecal = pyfits.open('WAVECAL_TABLE-01_flat.fits')
>>> muse_scibasic.calib.WAVECAL_TABLE = wavecal

To set more than one file for a tag, put the file names and/or :class:`pyfits.HDUList` objects into a
list.

Run the recipe with the default (first) raw data tag:

>>> res = muse_scibasic('Scene_fusion_1.fits')

Run the recipe with a nondefault tag (use raw data tag as argument name):

>>> res = muse_scibasic(SKY = 'sky_newmoon_no_noise_1.fits')

Run the recipe with alternative parameter or calibration tag setting (use
parameter names or calibration tags as argument names)

>>> res =  muse_scibasic('Scene_fusion_1.fits', nifu = 2, MASTER_FLAT = None,
...                      WAVECAL_TABLE = 'WAVECAL_TABLE_noflat.fits')

The results of a calibration run are :class:`pyfits.HDUList` objects.  To save them
(use output tags as attributes):

>>> res.PIXTABLE_OBJECT.writeto('Scene_fusion_pixtable.fits')

They can also be used directly as input of other recipes. 

>>> muse_sky = cpl.Recipe('muse_sky')
...
>>> res_sky = muse_sky(res.PIXTABLE_OBJECT)

If not saved, the output is usually lost! During recipe run, a temporary
directory is created where the :class:`pyfits.HDUList` input objects and the output files are
put into. This directory is cleaned up afterwards.

To control message verbosity on terminal (use :literal:`'debug'`,
:literal:`'info'`, :literal:`'warn'`, :literal:`'error'` or :literal:`'off'`):

>>> cpl.msg.level = 'debug'

To open a log file additionally to the output (possible only once!)

>>> cpl.msg.logfile('muse_pipeline.log', level = 'debug')
