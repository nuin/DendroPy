*************************
Reading Phylogenetic Data
*************************

Creating and Populating New Objects
===================================

The |Tree|, |TreeList|, |CharacterMatrix|-derived, and |DataSet| classes all support ":meth:`get_from_*()`" factory methods that allow for the simultaneous instantiation and population of the objects from a data source:

    :meth:`get_from_stream(src, schema, **kwargs)`
        Takes a file or file-like object opened for reading the data source as the first argument, and a string specifying the schema as the second.

    :meth:`get_from_path(src, schema, **kwargs)`
        Takes a string specifying the path to the the data source file as the first argument, and a string specifying the schema as the second.

    :meth:`get_from_string(src, schema, **kwargs)`
        Takes a string containing the source data as the first argument, and a string specifying the schema as the second.

All these methods minimally take a source and :ref:`schema specification string <Specifying_the_Data_Source_Format>` as arguments and return a new object of the given type populated from the given source::

    >>> import dendropy
    >>> tree1 = dendropy.Tree.get_from_string("((A,B),(C,D))", schema="newick")
    >>> tree_list1 = dendropy.TreeList.get_from_path("pythonidae.mcmc.nex", schema="nexus")
    >>> dna1 = dendropy.DnaCharacterMatrix.get_from_stream(open("pythonidae.fasta"), "dnafasta")
    >>> std1 = dendropy.StandardCharacterMatrix.get_from_path("python_morph.nex", "nexus")
    >>> dataset1 = dendropy.DataSet.get_from_path("pythonidae.nex", "nexus")

The :ref:`schema specification string <Specifying_the_Data_Source_Format>` can be one of: "``nexus``", "``newick``", "``nexml``", "``fasta``", or "``phylip``". Not all formats are supported for reading, and not all formats make sense for particular objects (for example, it would not make sense to try and instantiate a |Tree| or |TreeList| object from a FASTA-formatted data source).

Alternatively, you can also pass a file-like object and a :ref:`schema specification string <Specifying_the_Data_Source_Format>` to the constructor of these classes using the keyword arguments ``stream`` and ``schema`` respectively::

    >>> import dendropy
    >>> tree1 = dendropy.Tree(stream=open("mle.tre"), schema="newick")
    >>> tree_list1 = dendropy.TreeList(stream=open("pythonidae.mcmc.nex"), schema="nexus")
    >>> dna1 = dendropy.DnaCharacterMatrix(stream=open("pythonidae.fasta"), schema="dnafasta")
    >>> std1 = dendropy.StandardCharacterMatrix(stream=open("python_morph.nex"), schema="nexus")
    >>> dataset1 = dendropy.DataSet(stream=open("pythonidae.nex"), schema="nexus")

Various :ref:`keyword arguments <Customizing_Data_Creation_and_Reading>` can also be passed to these methods which customize or control how the data is parsed and mapped into DendroPy object space. These are discussed :ref:`below <Customizing_Data_Creation_and_Reading>`.

Reading and Populating (or Repopulating) Existing Objects
=========================================================

The |Tree|, |TreeList|, |CharacterMatrix|-derived, and |DataSet| classes all support a suite of ":meth:`read_from_*()`" instance methods that parallels the ":meth:`get_from_*()`" factory methods described above:

    :meth:`read_from_stream(src, schema, **kwargs)`
        Takes a file or file-like object opened for reading the data source as the first argument, and a string specifying the schema as the second.

    :meth:`read_from_path(src, schema, **kwargs)`
        Takes a string specifying the path to the the data source file as the first argument, and a string specifying the schema as the second.

    :meth:`read_from_string(src, schema, **kwargs)`
        Takes a string specifying containing the source data as the first argument, and a string specifying the schema as the second.

When called on an existing |TreeList| or |DataSet| object, these methods *add* the data from the data source to the object, whereas when called on an existing |Tree| or |CharacterMatrix| object,  they *replace* the object's data with data from the data source.
As with the ":meth:`get_from_*()`" methods, the :ref:`schema specification string <Specifying_the_Data_Source_Format>` can be any supported and type-apppropriate schema, such as "``nexus``", "``newick``", "``nexml``", "``fasta``", "``phylip``", etc.

For example, the following accumulates post-burn-in trees from several different files into a single |TreeList| object::

    >>> import dendropy
    >>> post_trees = dendropy.TreeList()
    >>> post_trees.read_from_path("pythonidae.nex.run1.t", "nexus", tree_offset=200)
    >>> print(post_trees.description())
    TreeList object at 0x550990 (TreeList5573008): 801 Trees
    >>> post_trees.read_from_path("pythonidae.nex.run2.t", "nexus", tree_offset=200)
    >>> print(post_trees.description())
    TreeList object at 0x550990 (TreeList5573008): 1602 Trees
    >>> post_trees.read_from_path("pythonidae.nex.run3.t", "nexus", tree_offset=200)
    >>> print(post_trees.description())
    TreeList object at 0x550990 (TreeList5573008): 2403 Trees
    >>> post_trees.read_from_path("pythonidae.nex.run4.t", "nexus", tree_offset=200)
    >>> print(post_trees.description())
    TreeList object at 0x5508a0 (TreeList5572768): 3204 Trees

The |TreeList| object automatically handles taxon management, and ensures that all appended |Tree| objects share the same |TaxonSet| reference. Thus all the |Tree| objects created and aggregated from the data sources in the example will all share the same |TaxonSet| and |Taxon| objects, which is important if you are going to be carrying comparisons or operations between multiple |Tree| objects.

In contrast to the aggregating behavior of :meth:`read_from_*()` of |TreeList| and |DataSet| objects, the :meth:`read_from_*()` methods of |Tree|- and |CharacterMatrix|-derived objects show replacement behavior. For example, the following changes the contents of a |Tree| by re-reading it::

    >>> import dendropy
    >>> t = dendropy.Tree()
    >>> t.read_from_path('pythonidae.mle.nex', 'nexus')
    >>> print(t.description())
    Tree object at 0x79c70 (Tree37413776: '0'): ('Python molurus':0.0779719244,(('Python sebae':0.1414715009,((((('Morelia tracyae':0.0435011998,('Morelia amethistina':0.0305993564,(('Morelia nauta':0.0092774432,'Morelia kinghorni':0.0093145395):0.005595,'Morelia clastolepis':0.005204698):0.023435):0.012223):0.025359,'Morelia boeleni':0.0863199106):0.019894,(('Python reticulatus':0.0828549023,'Python timoriensis':0.0963051344):0.072003,'Morelia oenpelliensis':0.0820543043):0.002785):0.00274,(((('Morelia viridis':0.0925974416,('Morelia carinata':0.0943697342,('Morelia spilota':0.0237557178,'Morelia bredli':0.0357358071):0.041377):0.005225):0.004424,('Antaresia maculosa':0.1141193265,(('Antaresia childreni':0.0363195704,'Antaresia stimsoni':0.0188535952):0.043287,'Antaresia perthensis':0.0947695442):0.019148):0.007921):0.022413,('Leiopython albertisii':0.0698883547,'Bothrochilus boa':0.0811607602):0.020941):0.007439,(('Liasis olivaceus':0.0449896545,('Liasis mackloti':0.0331564496,'Liasis fuscus':0.0230286886):0.058253):0.016766,'Apodora papuana':0.0847328612):0.008417):0.006539):0.011557,('Aspidites ramsayi':0.0349772256,'Aspidites melanocephalus':0.0577536309):0.042499):0.036177):0.016859,'Python brongersmai':0.1147218285):0.001271,'Python regius':0.1800489093):0.0
    >>> t.read_from_path('pythonidae.mcmc-con.nex', 'nexus')
    >>> print(t.description())
    Tree object at 0x79c70 (Tree37414064: 'con 50 majrule'): ('Python regius':0.212275,('Python sebae':0.176816,(((((('Antaresia maculosa':0.127351,('Antaresia perthensis':0.108378,('Antaresia stimsoni':0.021372,'Antaresia childreni':0.038155):0.046446):0.025262):0.012957,('Morelia carinata':0.101145,('Morelia bredli':0.038563,'Morelia spilota':0.025643):0.050967):0.010472,'Morelia viridis':0.098541):0.023291,('Bothrochilus boa':0.091928,'Leiopython albertisii':0.080986):0.031583):0.008347,((('Liasis fuscus':0.026601,'Liasis mackloti':0.034524):0.069881,'Liasis olivaceus':0.047727):0.023758,'Apodora papuana':0.096097):0.01474):0.010084,(('Python timoriensis':0.101865,'Python reticulatus':0.095018):0.0922,('Morelia boeleni':0.093309,('Morelia tracyae':0.04727,('Morelia amethistina':0.034936,(('Morelia nauta':0.011,'Morelia kinghorni':0.011198):0.006932,'Morelia clastolepis':0.008103):0.025987):0.017415):0.033886):0.027519,'Morelia oenpelliensis':0.092143):0.006779):0.018238,('Aspidites ramsayi':0.030898,'Aspidites melanocephalus':0.068553):0.049525):0.050607):0.023304,('Python brongersmai':0.132193,'Python molurus':0.08872):0.011466)

As with the :meth:`get_from_*()` methods, keyword arguments can be used to provide :ref:`control on the data source parsing <Customizing_Data_Creation_and_Reading>`.

.. _Specifying_the_Data_Source_Format:

Specifying the Data Source Format
==================================

All the :meth:`get_from_*()` and :meth:`read_from_*()` methods take a schema specification string using the ``schema`` argument which specifies the format of the data source.

The string can be one of the following:

    "``nexus``"
        To read |Tree|, |TreeList|, |CharacterMatrix|, or |DataSet| objects from a NEXUS-formatted source.

    "``newick``"
        To read |Tree|, |TreeList|, or |DataSet| objects from a Newick-formatted source.

    "``fasta``"
        To read |CharacterMatrix| or |DataSet| objects from a FASTA-formatted source. FASTA-sources require the additional keyword, ``data_type``, that describes the type of data: "``dna``", "``rna``", "``protein``", "``standard``"" (discrete data represented as binary 0/1), "``restriction``" (restriction sites), or "``infinite``" (infinite sites).

    "``phylip``"
        To read |CharacterMatrix| or |DataSet| objects from a PHYLIP-formatted source. PHYLIP-sources require the additional keyword, ``data_type``, that describes the type of data: "``dna``", "``rna``", "``protein``", "``standard``"" (discrete data represented as binary 0/1), "``restriction``" (restriction sites), or "``infinite``" (infinite sites).

.. _Customizing_Data_Creation_and_Reading:

Customizing Data Creation and Reading
=====================================

When specifying a data source from which to create or populate data objects using the :meth:`get_from_*()`, :meth:`read_from_*()`, or passing a data source stream to a constructor, you can also specify keyword arguments that provide fine-grained control over how the data source is parsed.
Some of these keyword arguments apply generally, regardless of the format of the data source or the data object being created, while others are specific to the data object type or the data source format.

.. _Customizing_Reading_All_Formats:

All Formats
^^^^^^^^^^^

    ``attached_taxon_set``
        If :keyword:`True` when reading into a |DataSet| object, then a new |TaxonSet| object will be created and added to the :attr:`~dendropy.dataobject.dataset.DataSet.taxon_sets` list of the |DataSet| object, and the |DataSet| object will be placed in "attached" (or single) taxon set mode, i.e., all taxa in any data sources parsed or read will be mapped to the same |TaxonSet| object. By default, this is :keyword:`False`, resulting in a multi-taxon set mode |DataSet| object.

    ``taxon_set``
        If passed a |TaxonSet| object, then this |TaxonSet| will be used to manage all taxon references in the data source.
        When creating a new |Tree|, |TreeList| or |CharacterMatrix| object from a data source, the |TaxonSet| object passed by this keyword will be used as the |TaxonSet| associated with the object.
        When reading into a |DataSet| object, if the data source defines multiple collections of taxa (as is possible with, for example, the NEXML schema, or the Mesquite variant of the NEXUS schema), then multiple new |TaxonSet| object will be created. By passing a |TaxonSet| object through the ``taxon_set`` keyword, you can force DendroPy to use the same |TaxonSet| object for all taxon references.

    ``exclude_trees``
        If :keyword:`True`, then all tree data in the data source will be skipped.
        Default value is :keyword:`False`, i.e., all tree data will be included.

    ``exclude_chars``
        If :keyword:`True`, then all character data in the data source will be skipped.
        Default value is :keyword:`False`, i.e., all character data will be included.

.. _Customizing_Reading_NEXUS_and_Newick:

NEXUS/Newick
^^^^^^^^^^^^

    ``is_rooted``, ``is_unrooted``, ``default_as_rooted``, ``default_as_unrooted``

        When reading into a |Tree|, |TreeList|, or |DataSet| object, this keyword determines how trees in the data source will be rooted.
        The rooting state of a |Tree| object is set by the :attr:`~dendropy.dataobject.tree.Tree.is_rooted` property.
        When parsing NEXUS- and Newick-formatted data, the rooting states of the resulting |Tree| objects are given by ``[&R]`` (for rooted) or ``[&U]`` (for unrooted) comment tags preceding the tree definition in the data source.
        If these tags are not present, then the trees are assumed to be unrooted.
        This behavior can be changed by specifying keyword arguments to the :meth:`get_from_*()`,  or :meth:`read_from_*()` methods of both the |Tree| and |TreeList| classes, or the constructors of these classes when specifying a data source from which to construct the tree:

        The ``as_rooted`` keyword argument, if :keyword:`True`, forces all trees to be interpreted as rooted, regardless of whether or not the ``[&R]``/``[&U]`` comment tags are given.
        Conversely, if :keyword:`False`, all trees will be interpreted as unrooted.
        For semantic clarity, you can also specify ``as_unrooted`` to be :keyword:`True` to force all trees to be unrooted.

        .. literalinclude:: /examples/tree_rootings1.py
            :linenos:

        In addition, you can specify a ``default_as_rooted`` keyword argument, which, if :keyword:`True`, forces all trees to be interpreted as rooted, *if* the ``[&R]``/``[&U]`` comment tags are *not* given.
        Otherwise the rooting will follow the ``[&R]``/``[&U]`` commands.
        Conversely, if ``default_as_rooted`` is :keyword:`False`, all trees will be interpreted as unrooted if the ``[&R]``/``[&U]`` comment tags are not given.
        Again, for semantic clarity, you can also specify ``default_as_unrooted`` to be :keyword:`True` to assume all trees are unrooted if not explicitly specified, though, as this is default behavior, this should not be neccessary.

    ``preserve_underscores``

        With NEXUS and Newick data sources, you can also specify ``preserve_underscores=True``.
        The NEXUS standard dictates that underscores are equivalent to spaces, and thus any underscore found in any unquoted label in a NEXUS/Newick data source will be substituted for spaces.
        Specifying ``preserve_underscores=True`` will force DendroPy to keep the underscores.

.. _Customizing_Reading_PHYLIP:

PHYLIP
^^^^^^

    ``data_type``
        As noted above, the PHYLIP format requires specification of the type of data using the ``data_type`` argument, which takes one of the following strings: "``dna``", "``rna``", "``protein``", "``standard``"", "``restriction``", or "``infinite``".

    ``strict``
        By default, the PHYLIP parser works in "relaxed" mode, which means that taxon labels can be of arbitrary length, and taxon labels and corresponding sequences are separated by one or more spaces. By specifying ``strict=True``, the parse will behave in strict mode, i.e., where taxon labels are limited to 10 characters in length, and sequences start on column 11.

    ``interleaved``
        By default, the PHYLIP parsers assumes that the data source is in sequential format. If the data is in interleaved format, you should specify ``interleaved=True``.

    ``multispace_delimiter``
        The default "relaxed" mode of the PHYLIP parser assumes that taxon labels are separated from sequence characters by one or more spaces. By specifying ``multispace_delimiter=True``, the parser will require two or more spaces to separate taxon labels from sequence characters, thus allowing you to use single spaces in your taxon labels.

    ``ignore_invalid_chars``
        By default, the PHYLIP parser will fail with an error if invalid characters are found in a sequence. By specifying ``ignore_invalid_chars=True``, the parser will simply ignore these characters.

    ``underscores_to_spaces``
        In the default relaxed PHYLIP format mode, since the first occurrence of a space in the data format is taken to denote the end of the taxon label, spaces are not permitted within taxon labels. A common convention is to use underscores in place of spaces in cases like this. By specifying ``underscores_to_spaces=True``, the parser will automatically substitute any underscores found in taxon labels with spaces, thus allowing for correspondence with the same taxa represented in other formats that allow spaces, such as NEXUS or Newick.
