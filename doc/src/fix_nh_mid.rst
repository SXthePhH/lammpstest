.. index:: fix nvt/mid
.. index:: fix npt/mid
.. index:: fix nph/mid

fix nvt/mid command
===================

fix npt/mid command
===================

fix nph/mid command
===================

Syntax
""""""

.. code-block:: LAMMPS

   fix ID group-ID style_name keyword value ...

* ID, group-ID are documented in :doc:`fix <fix>` command
* style_name = *nvt/mid* or *npt/mid* or *nph/mid*
* one or more keyword/value pairs may be appended

  .. parsed-literal::

     keyword = *temp* or *iso* or *aniso* or *tri* or *x* or *y* or *z* or *xy* or *yz* or *xz* or *couple* or *tchain* or *pchain* or *mtk* or *tloop* or *ploop* or *nreset* or *drag* or *ptemp* or *dilate* or *scalexy* or *scaleyz* or *scalexz* or *flip* or *fixedpoint* or *thermostat* or *barostat* or *integrator* or *seed* or *zero*
       *temp* values = Tstart Tstop Tdamp
         Tstart,Tstop = external temperature at start/end of run
         Tdamp = Nose-Hoover temperature damping parameter (time units)
       *iso* or *aniso* or *tri* values = Pstart Pstop Pdamp
         Pstart,Pstop = scalar external pressure at start/end of run (pressure units)
         Pdamp = Nose-Hoover pressure damping parameter (time units)
       *x* or *y* or *z* or *xy* or *yz* or *xz* values = Pstart Pstop Pdamp
         Pstart,Pstop = external stress tensor component at start/end of run (pressure units)
         Pdamp = stress damping parameter (time units)
       *couple* = *none* or *xyz* or *xy* or *yz* or *xz*
       *thermostat* value = *nh* or *langevin*
         nh = use a Nose-Hoover chain for the particle thermostat
         langevin = use a Langevin O-step for the particle thermostat
       *thermostat* values = *langevin* Tdamp
         Tdamp = Langevin particle thermostat damping parameter (time units)
       *barostat* value = *nh* or *langevin*
         nh = use a Nose-Hoover chain for the barostat thermostat
         langevin = use a Langevin O-step for the barostat thermostat
       *barostat* values = *langevin* Pdamp
         Pdamp = Langevin barostat thermostat damping parameter (time units)
       *integrator* value = *middle* or *side*
         middle = apply thermostat/barostat O-steps in the middle of the time step
         side = apply thermostat/barostat O-steps at the sides of the time step
       *seed* value = N
         N = random number seed for Langevin thermostatting
       *zero* value = *yes* or *no*
         yes = remove the mass-weighted center-of-mass part of particle Langevin random kicks
       *tchain*, *pchain*, *mtk*, *tloop*, *ploop*, *nreset*, *drag*, *ptemp*,
       *dilate*, *scalexy*, *scaleyz*, *scalexz*, *flip*, and *fixedpoint*
       have the same meaning as for :doc:`fix nvt, npt, and nph <fix_nh>`

Examples
""""""""

.. code-block:: LAMMPS

   fix 1 all nvt/mid temp 300.0 300.0 100.0
   fix 2 all nvt/mid temp 300.0 300.0 100.0 thermostat langevin 100.0 seed 492845
   fix 3 all npt/mid temp 300.0 300.0 100.0 iso 1.0 1.0 1000.0
   fix 4 all npt/mid temp 300.0 300.0 100.0 aniso 1.0 1.0 1000.0 couple xz barostat langevin 1000.0 seed 928374
   fix 5 all nph/mid tri 1.0 1.0 1000.0 ptemp 300.0 integrator side

Description
"""""""""""

These commands perform constant NVT, NPT, or NPH time integration using
the same Nose-Hoover thermostat and barostat variables as the
:doc:`fix nvt, npt, and nph <fix_nh>` commands, but with a selectable
operator ordering intended for middle-scheme integration.  The
*nvt/mid* style thermostats particle velocities, the *nph/mid* style
barostats the simulation cell without particle thermostatting, and the
*npt/mid* style performs both particle thermostatting and barostatting.

By default these fixes use Nose-Hoover chains for both the particle
thermostat and the barostat thermostat, matching the corresponding
standard fix styles as closely as possible except for the time-splitting
order.  The *thermostat langevin* and *barostat langevin* options replace
the corresponding Nose-Hoover chain update by a stochastic Langevin
O-step.  When a Langevin option is used, the *seed* keyword selects the
random number stream.  The same seed is used for particle and barostat
Langevin steps if both are enabled.

The *integrator* keyword selects where the thermostat and barostat
O-steps are applied within the velocity-Verlet time step.  The default
is *middle*.  The *side* option applies those O-steps at the sides of the
time step and is provided for comparisons and algorithmic testing.

The *zero* keyword affects the particle Langevin thermostat.  If *zero
yes* is used, the mass-weighted center-of-mass part of the random
particle velocity kick is removed so that the random thermostat step
does not inject net linear momentum.  If *zero no* is used, no such
correction is applied.

All pressure-control keywords, pressure coupling modes, triclinic-cell
options, MTK correction settings, and dilation options are inherited from
:doc:`fix nvt, npt, and nph <fix_nh>`.  For example, *iso*, *aniso*,
*tri*, *x*, *y*, *z*, *xy*, *xz*, *yz*, and *couple* select the
barostatted pressure components in the same manner as for *fix npt* and
*fix nph*.

The temperature and pressure computes created by these fixes follow the
same convention as the standard fixes.  The *nvt/mid* style creates a
compute named fix-ID + ``_temp`` for the fix group.  The *npt/mid* and
*nph/mid* styles create a compute named fix-ID + ``_temp`` for group
``all`` and a pressure compute named fix-ID + ``_press`` that uses that
temperature compute.  The pressure is therefore computed for the entire
system, even when the integration fix acts on a subset of atoms.  This
is the same convention as :doc:`fix npt <fix_nh>` and
:doc:`fix nph <fix_nh>`.

Restart, fix_modify, output, run start/stop, minimize info
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

These fixes write their thermostat, barostat, middle-integrator, and
random-number state to :doc:`binary restart files <restart>`.  This
includes Nose-Hoover chain variables, barostat variables, the selected
middle or side ordering, Langevin damping choices, and the stochastic
random number generator state.  See the :doc:`read_restart
<read_restart>` command for information on how to re-specify a fix in an
input script that reads a restart file.

The operation of these fixes should continue in an uninterrupted fashion
when the same fix ID and style are re-specified after reading a restart
file.  Any fix keywords that define the desired ensemble should also be
specified consistently in the restarted input script.

The :doc:`fix_modify <fix_modify>` *temp* and *press* options have the
same meaning and restrictions as for the standard Nose-Hoover fixes.

These fixes compute the same global scalar and global vector quantities
as :doc:`fix nvt, npt, and nph <fix_nh>`.  The scalar is the cumulative
energy change imposed by thermostatting and/or barostatting and is
extensive.  The vector stores the thermostat and barostat internal
variables and is intensive.

These fixes can ramp their external temperature and pressure over
multiple runs using the *start* and *stop* keywords of the :doc:`run
<run>` command.

These fixes are not invoked during :doc:`energy minimization <minimize>`.

Restrictions
""""""""""""

The restrictions for :doc:`fix nvt, npt, and nph <fix_nh>` also apply.
In particular, barostatted dimensions must be periodic, off-diagonal
stress components require a triclinic simulation box, and the final
temperature for the *temp* keyword cannot be 0.0.

The *nvt/mid* style requires the *temp* keyword and cannot be used with
pressure-control keywords.  The *nph/mid* style requires pressure
control and cannot be used with the *temp* keyword.  The *npt/mid* style
requires both the *temp* keyword and pressure control.

The *thermostat langevin* option is only meaningful when particle
thermostatting is active.  The *barostat langevin* option is only
meaningful when pressure control is active.

Related commands
""""""""""""""""

:doc:`fix nvt <fix_nh>`, :doc:`fix nph <fix_nh>`,
:doc:`fix npt <fix_nh>`, :doc:`fix_modify <fix_modify>`,
:doc:`run_style <run_style>`

Default
"""""""

The keyword defaults are thermostat = nh, barostat = nh, integrator =
middle, seed = 12345678, zero = yes.  Other defaults are the same as for
:doc:`fix nvt, npt, and nph <fix_nh>`: tchain = 3, pchain = 3, mtk =
yes, tloop = 1, ploop = 1, nreset = 0, drag = 0.0, dilate = all, couple
= none, flip = yes, and the tilt scaling defaults documented for the
standard Nose-Hoover fixes.
