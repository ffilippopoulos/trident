.. _deploying-in-kubernetes:

#################
Deploying Trident
#################

This guide will take you through the process of deploying Trident for the
first time and provisioning your first volume automatically. If you are a
new user, this is the place to get started with using Trident.

If you are an existing user looking to upgrade, head on over to the
:ref:`Upgrading Trident <Upgrading Trident>` section.

There are two ways you can deploy Trident:

1. **Using the Trident Operator:** Trident now provides a
   `Kubernetes Operator <https://kubernetes.io/docs/concepts/extend-kubernetes/operator/>`_
   to deploy Trident. The Trident Operator controls the installation of
   Trident, taking care to **self-heal the install and manage changes as
   well as upgrades to the Trident installation**. Take a look at
   :ref:`Deploying with the Trident Operator <deploying-with-operator>`!

   .. note::
      Starting with Trident 21.01, you can use Helm to install Trident Operator.

2. **Deploying Trident with tridentctl:** If you have already deployed
   previous releases, this is the method of deployment that you would have
   used. :ref:`This page <deploying-with-tridentctl>` explains all the steps
   involved in deploying Trident in this manner.

Choosing the right option
-------------------------

To determine which deployment option to use, you must consider the following:

Why should I use Helm?
~~~~~~~~~~~~~~~~~~~~~~

If you have other applications that you are managing using Helm, starting with Trident 21.01, you can manage your Trident installation also using Helm.

Why should I use the Trident Operator?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you are a new user testing Trident (or) deploying a fresh installation of
Trident in your cluster, the Trident Operator is a great way to dynamically
manage Trident resources and automate the setup phase. There are some
prerequisites that must be satisfied. Please refer to the :ref:`Requirements <Requirements>`
section to identify the necessary requirements to deploy with the Trident
Operator.

The Trident Operator offers a number of benefits such as:

Self-Healing
""""""""""""

The biggest advantage that the operator provides is
the ability to monitor a Trident installation and actively take measures
to address issues, such as when the Trident deployment is deleted or if
the installation is modified accidentally. When the operator is set
up as a deployment, a ``trident-operator-<generated-id>`` pod is created.
This pod associates a TridentOrchestrator CR with a Trident installation and always
ensures there exists only one active TridentOrchestrator. In other words, the
operator makes sure there's only one instance of Trident in the cluster and
controls its setup, making sure the installation is idempotent. When changes
are made to the Trident install [such as deleting the Trident deployment or
node daemonset], the operator identifies them and fixes them
individually.

Updating existing installations
"""""""""""""""""""""""""""""""

With the operator it is easy to update an existing Trident deployment. Since
the Trident install is initiated by the creation of a ``TridentOrchestrator``
CR, you can edit it to make updates to an already created Trident installation.
Wherein installations done with ``tridentctl`` will require an
uninstall/reinstall to perform something similar, the operator only requires
editing the TridentOrchestrator CR.

As an example, consider a scenario where you need to enable Trident to generate
debug logs. To do this, you will need to patch your TridentOrchestrator to set
``spec.debug`` to ``true``.

.. code-block:: console

   kubectl patch torc <trident-orchestrator-name> -n trident --type=merge -p '{"spec":{"debug":true}}'

After the TridentOrchestrator is updated, the operator processes the updates and
patches the existing installation. This may triggers the creation of new pods
to modify the installation accordingly.

Handling Kubernetes upgrades
""""""""""""""""""""""""""""

When the Kubernetes version of the cluster is upgraded to a
:ref:`supported version <Supported frontends (orchestrators)>`, the operator
updates an existing Trident installation automatically and changes it
to make sure it meets the requirements of the Kubernetes version.

If the cluster is upgraded to an unsupported version:

* the operator prevents installing Trident.

* If Trident has already been installed with the operator, a warning is
  displayed to indicate that Trident is installed on an unsupported Kubernetes
  version.

.. warning::

  If you are looking to install Trident using the operator/Helm Chart on
  OpenShift Container Platform, it is recommended that you install Trident v21.01.1
  or later. The Trident operator released with v21.01.0 contains a
  `known issue <https://github.com/NetApp/trident/issues/517>`_ that has been
  fixed with v21.01.1. For more details take a look at :ref:`Known Issues <2101-operator-bug>`.

When should I use tridentctl?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you have an existing Trident deployment that must be upgraded to or if
you are looking to highly customize your Trident install, you should take a
look at using ``tridentctl`` to setup Trident. This is the conventional method
of installing Trident. Take a look at the :ref:`Upgrading <Upgrading Trident>`
page to upgrade Trident.

Ultimately, the environment in question will determine the choice of deployment.

Moving between installation methods
-----------------------------------

It is not hard to imagine a scenario where moving between deployment methods is
desired. Here's what you must know before attempting to move from a ``tridentctl``
install to an operator-based deployment, or vice versa:

1. Always use the same method for uninstalling Trident. If you have deployed Trident
   with ``tridentctl``, you must use the appropriate version of the ``tridentctl``
   binary  to uninstall Trident. Similarly, if deploying Trident with the operator,
   you must edit the ``TridentOrchestrator`` CR and set ``spec.uninstall=true``
   to uninstall Trident.

2. If you have a Trident Operator deployment that you want to remove and use ``tridentctl``
   to deploy Trident, you must first edit the ``TridentOrchestrator`` and set
   ``spec.uninstall=true`` to uninstall Trident. You will then have delete the
   ``TridentOrchestrator`` and the operator deployment.
   You can then install Trident with ``tridentctl``.

3. If you have a manual Trident Operator deployment, and you want to use Helm-based Trident Operator deployment, you should manually uninstall the Trident Operator first, and then do the ``helm install``. This enables Helm to deploy the Trident Operator with the required labels and annotations. If you do not do this, your Helm-based Trident Operator deployment will fail with ``label validation error`` and ``annotation validation error``.
   If you have a ``tridentctl``-based installation, you can use Helm-based deployment without running into issues.

NetApp **does not recommend downgrading Trident releases** unless absolutely necessary.
