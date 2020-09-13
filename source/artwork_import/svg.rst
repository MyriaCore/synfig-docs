########################
    Importing SVG
########################


<!-- Page info -->
{{Title|SVG Import}}
{{Navigation|Category:Manual|Doc:ListImporter}}
{{Category|Manual}}
{{Category|ImportArt}}
{{TOCright}}
{{NewTerminology}}
<!-- Page info end -->
=Inkscape SIF exporter extension=

The [https://inkscape.org Inkscape] extension converts SVG files to Synfig Studio (.sif) format and it is a recommended way.

Since '''Inkscape 0.91''' the extension is shipped with the official Inkscape installation and can be found from - {{c|<File>|<Save as>|Animation synfig (.sif)}}.

If you want to manually install it, you can found it at [http://www.synfig.org/cms/en/download/tools/ Scripts & Tools] download section.

==Usage==
'''svg2sif''' adds a {{Literal|Synfig Animation (*.sif)}} option to Inkscape's {{Literal|Save As}} dialog.

I you want to group objects, put all the objects to group on their own layer in inkscape. This way, they end up in a {{l|Group Layer}} once in the sif format.

If your SVG document contains clones or non-paths (text, rectangles, etc.) the extension will quickly open another Inkscape window to convert these elements to paths. This may take a while. The file is not saved until the status bar reads {{Literal|Document Saved.}}

If you save frequently, you can speed up the process by selecting {{c|<Extensions>|Synfig|Prepare for Export}} from the Inkscape menu. This will convert everything in the current document to paths (but may make it harder to edit).

Source : [https://github.com/nikitakit/svg2sif/blob/master/README.md github/svg2sif/README.md]

=Import SVG directly in Synfig Studio=
'''Since version 0.91 of Inkscape (linux and windows) it is unnecessary to use additional extension, Inkscape [https://inkscape.org/en/learn/animation/#standalone-programs includes] the registration under the extension SIF .'''

From previous Synfig versions there is an option to import SVG from {{c|<File>|Import}} menu. This seems to work better than the options described below, however there may be problems importing some SVG elements correctly. See [http://synfig.org/forums/viewtopic.php?f=12&t=2728 this topic] in forums for some hints.

* First import may fail, try to import same file for two times.
* 0.62.02 is reported to work better in Ubuntu than version 0.63.00.

=Other options=
The following are outdated options or special options for experts.
== Option 1 ==
This uses an XSLT 2.0 stylesheet to transform SVG XML to Synfig XML.
=== Objective ===
Turn an SVG image into a Synfig file for import. First posted in the forums: http://synfig.org/forums/viewtopic.php?t=30
=== Prerequisites ===
# Make sure a Java runtime environment is installed.
# Get a recent version of the SAXON XSLT processor for Java from http://saxon.sourceforge.net/. Recommended version: Saxon-SA 9.0 (saxonsa9-0-0-2j.zip).
# Extract the SAXON package to a folder of your choice. As an example we're going to use d:\saxon in the following. The folder will contain several JAR files.
# Create the file d:\saxon\svg2synfig.xsl with the content provided at the bottom of this document ({{l|#svg2synfig.xsl}}).
==== Optional Prerequites for Windows ====
If you don't want to use the command line, create a batch file d:\saxon\svg2synfig.bat with this content:
 @java -jar %0\..\saxon9.jar -xsl:%0\..\svg2synfig.xsl %1 > %0\..\synfig.sif
 @pause
=== Transforming an SVG into a Synfig File ===
==== Windows Batch File ====
# Just drop the SVG file onto svg2synfig.bat.
==== Command line ====
# Change directory to d:\saxon.
# Enter the following command (replace your_input.svg by the path of the SVG file):
 java -jar saxon9.jar -xsl:svg2synfig.xsl your_input.svg > synfig.sif
=== Result ===
If the conversion has been successful, the result will be written to the file d:\saxon\synfig.sif.
You can open this file in Synfig.
=== Limitations ===
* Doesn't seem to work with SaxonB (FOSS version)
* Compressed SVG (svgz) must be uncompressed first.
* Only SVG path objects are supported. Try converting all objects to paths.
* Only a subset of path elements is supported. Try to modify all path nodes to have split tangents, and all path segments to be curves.
* Sophisticated coloring (e. g. gradients) is not supported.
* Only basic transformations are supported.
* Fill and outline on the same object is not supported.
=== svg2synfig.xsl ===
<pre><nowiki>
<xsl:stylesheet version="2.0" exclude-result-prefixes="#all"
		xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
		xmlns:svg="http://www.w3.org/2000/svg"
		xmlns:xs="http://www.w3.org/2001/XMLSchema"
		xmlns:math="http://exslt.org/math">
	<xsl:output method="xml" indent="yes" encoding="UTF-8"/>

	<xsl:template match="/">
		<xsl:apply-templates/>
	</xsl:template>

	<xsl:template match="svg:svg">
		<xsl:variable name="width" select="math:units_to_px(@width)"/>
		<xsl:variable name="height" select="math:units_to_px(@height)"/>
		<xsl:variable name="has_view_box" select="matches(@viewBox, '(\d+\s){3}\d+')"/>
		<canvas version="0.2" id="{@id}"
				width="{if ($has_view_box) then replace(@viewBox, '(\d+)\s(\d+)\s(\d+)\s(\d+)', '$3') else $width}"
				height="{if ($has_view_box) then replace(@viewBox, '(\d+)\s(\d+)\s(\d+)\s(\d+)', '$4') else $height}"
				view-box="{if ($has_view_box) then @viewBox else concat('0 0 ', $width, ' ', $height)}">
			<xsl:apply-templates select="svg:g|svg:svg|svg:path"/>
		</canvas>
	</xsl:template>

	<xsl:template match="svg:g">
		<layer type="PasteCanvas" active="true" version="0.1" desc="{@id}">
			<param name="canvas">
				<canvas>
					<xsl:apply-templates/>
				</canvas>
			</param>
		</layer>
	</xsl:template>

	<xsl:template match="svg:path">
		<xsl:variable name="style">
			<xsl:for-each select="ancestor-or-self::*">
				<xsl:sort select="position()" data-type="number" order="descending"/>
				<xsl:value-of select="concat(@style, ';fill:', @fill, ';stroke:', @stroke, ';stroke-width:', @stroke-width, ';')"/>
			</xsl:for-each>
		</xsl:variable>
		<xsl:variable name="self" select="."/>
		<xsl:variable name="is_fill" select="not(matches(replace($style, 'fill:[^n;][^o].*', ''), 'fill:none'))"/>
		<xsl:analyze-string select="@d" regex="m[^z]+(z|$)" flags="i">
			<xsl:matching-substring>
				<layer type="{if ($is_fill) then 'region' else 'outline'}" version="0.1" desc="{$self/@id}">
					<xsl:call-template name="style-to-color">
						<xsl:with-param name="style" select="replace(replace($style, ':none.*', ''), if ($is_fill) then '.*fill:([^;]+).*' else '.*stroke:([^;]+).*', '$1')"/>
					</xsl:call-template>
					<xsl:if test="not ($is_fill)">
						<xsl:call-template name="style-to-width">
							<xsl:with-param name="style" select="replace($style, '.*stroke-width:([^;]+).*', '$1')"/>
						</xsl:call-template>
					</xsl:if>
					<param name="bline">
						<bline type="bline_point" loop="{matches(., 'z', 'i')}">
							<xsl:call-template name="path-to-bline">
								<xsl:with-param name="path" select="."/>
								<xsl:with-param name="node" select="$self" tunnel="yes"/>
							</xsl:call-template>
						</bline>
					</param>
				</layer>
			</xsl:matching-substring>
		</xsl:analyze-string>
	</xsl:template>

	<xsl:template name="path-to-bline">
		<xsl:param name="path"/>
		<xsl:variable name="stripped" select="replace(replace(translate($path, ',', ' '), '(\d)-', '$1 -'), '\s*([a-z]+)\s*', '$1', 'i')"/>
		<xsl:variable name="closed" select="if (matches($stripped, 'z', 'i')) then $stripped else replace($stripped, 'm([-\d.]+\s[-\d.]+).*$', '$0l$1z', 'i')"/>
		<xsl:variable name="tmp" select="replace($closed, '([-\d.]+\s[-\d.]+)l([-\d.]+\s[-\d.]+)', '$1c$1 $2 $2', 'i')"/>
		<xsl:variable name="curve" select="replace($tmp, '([-\d.]+\s[-\d.]+)l([-\d.]+\s[-\d.]+)', '$1c$1 $2 $2', 'i')"/>
		<xsl:analyze-string select="$curve" regex="\s([-\d.]+\s[-\d.]+)\s[-\d.]+\s[-\d.]+z" flags="i">
			<xsl:matching-substring>
				<xsl:analyze-string select="concat(regex-group(1), $curve)" regex="([-\d.]+)\s([-\d.]+)[m\s]([-\d.]+)\s([-\d.]+)c([-\d.]+)\s([-\d.]+)" flags="i">
					<xsl:matching-substring>
						<xsl:call-template name="node-to-bline-point">
							<xsl:with-param name="c1_x" select="regex-group(1)"/>
							<xsl:with-param name="c1_y" select="regex-group(2)"/>
							<xsl:with-param name="x" select="regex-group(3)"/>
							<xsl:with-param name="y" select="regex-group(4)"/>
							<xsl:with-param name="c2_x" select="regex-group(5)"/>
							<xsl:with-param name="c2_y" select="regex-group(6)"/>
						</xsl:call-template>
					</xsl:matching-substring>
				</xsl:analyze-string>
			</xsl:matching-substring>
		</xsl:analyze-string>
	</xsl:template>

	<xsl:template name="node-to-bline-point">
		<xsl:param name="x"/>
		<xsl:param name="y"/>
		<xsl:param name="c1_x"/>
		<xsl:param name="c1_y"/>
		<xsl:param name="c2_x"/>
		<xsl:param name="c2_y"/>
		<xsl:param name="node" tunnel="yes"/>
		<xsl:variable name="transform">
			<xsl:for-each select="$node/ancestor-or-self::*/@transform">
				<xsl:value-of select="."/>
			</xsl:for-each>
		</xsl:variable>
		<xsl:variable name="t" select="math:resolve_transform($transform)"/>
		<xsl:variable name="transformed_x" select="$t[5] + $t[1] * xs:float($x) + $t[3] * xs:float($y)"/>
		<xsl:variable name="transformed_y" select="$t[6] + $t[2] * xs:float($x) + $t[4] * xs:float($y)"/>
		<xsl:variable name="transformed_c1_x" select="$t[5] + $t[1] * xs:float($c1_x) + $t[3] * xs:float($c1_y)"/>
		<xsl:variable name="transformed_c1_y" select="$t[6]+ $t[2] * xs:float($c1_x) + $t[4] * xs:float($c1_y)"/>
		<xsl:variable name="transformed_c2_x" select="$t[5] + $t[1] * xs:float($c2_x) + $t[3] * xs:float($c2_y)"/>
		<xsl:variable name="transformed_c2_y" select="$t[6]+ $t[2] * xs:float($c2_x) + $t[4] * xs:float($c2_y)"/>
		<entry>
			<composite type="bline_point">
				<point>
					<vector>
						<x><xsl:value-of select="$transformed_x"/></x>
						<y><xsl:value-of select="$transformed_y"/></y>
					</vector>
				</point>
				<width>
					<real value="1"/>
				</width>
				<origin>
					<real value="0.5"/>
				</origin>
				<split>
					<bool value="true"/>
				</split>
				<t1>
					<xsl:call-template name="vector-pair-to-radial">
						<xsl:with-param name="origin-x" select="$transformed_c1_x"/>
						<xsl:with-param name="origin-y" select="$transformed_c1_y"/>
						<xsl:with-param name="x" select="$transformed_x"/>
						<xsl:with-param name="y" select="$transformed_y"/>
					</xsl:call-template>
				</t1>
				<t2>
					<xsl:call-template name="vector-pair-to-radial">
						<xsl:with-param name="origin-x" select="$transformed_x"/>
						<xsl:with-param name="origin-y" select="$transformed_y"/>
						<xsl:with-param name="x" select="$transformed_c2_x"/>
						<xsl:with-param name="y" select="$transformed_c2_y"/>
					</xsl:call-template>
				</t2>
			</composite>
		</entry>
	</xsl:template>

	<xsl:template name="vector-pair-to-radial">
		<xsl:param name="x"/>
		<xsl:param name="y"/>
		<xsl:param name="origin-x"/>
		<xsl:param name="origin-y"/>
		<xsl:variable name="dx" select="xs:float($x) - xs:float($origin-x)"/>
		<xsl:variable name="dy" select="xs:float($y) - xs:float($origin-y)"/>
		<xsl:variable name="d" select="math:sqrt($dx * $dx + $dy * $dy)"/>
		<xsl:variable name="angle" select="math:atan2($dy, $dx)"/>
		<radial_composite type="vector">
			<radius>
				<real value="{$d * 3}"/>
			</radius>
			<theta>
				<angle value="{$angle * 57.295779513082320876798154814105}"/>
			</theta>
		</radial_composite>
	</xsl:template>

	<xsl:template name="style-to-width">
		<xsl:param name="style"/>
		<xsl:if test="matches($style, '^\d')">
			<param name="width">
				<real value="{math:units_to_px($style)}"/>
			 </param>
		</xsl:if>
	</xsl:template>

	<xsl:template name="style-to-color">
		<xsl:param name="style"/>
		<xsl:if test="matches($style, '#')">
			<xsl:analyze-string select="concat($style, ';')" regex="#([\da-f]{{2}})([\da-f]{{2}})([\da-f]{{2}});">
				<xsl:matching-substring>
					<param name="color">
						<color>
							<r><xsl:value-of select="math:hex_to_color(regex-group(1))"/></r>
							<g><xsl:value-of select="math:hex_to_color(regex-group(2))"/></g>
							<b><xsl:value-of select="math:hex_to_color(regex-group(3))"/></b>
							<a><xsl:value-of select="if (matches($style, 'fill-opacity:')) then math:power(xs:float(replace($style, '.*fill-opacity:([-\d.]+).*', '$1')), 1 div 2.2) else 1"/></a>
						</color>
					</param>
				</xsl:matching-substring>
			</xsl:analyze-string>
		</xsl:if>
		<xsl:if test="matches($style, 'rgb')">
			<xsl:analyze-string select="concat($style, ';')" regex="rgb[(\s]+([-\d.]+)[,\s]+([-\d.]+)[,\s]+([-\d.]+)[\s)]+;">
				<xsl:matching-substring>
					<param name="color">
						<color>
							<r><xsl:value-of select="math:power(xs:float(regex-group(1)) div 255, 2.2)"/></r>
							<g><xsl:value-of select="math:power(xs:float(regex-group(2)) div 255, 2.2)"/></g>
							<b><xsl:value-of select="math:power(xs:float(regex-group(3)) div 255, 2.2)"/></b>
							<a>1</a>
						</color>
					</param>
				</xsl:matching-substring>
			</xsl:analyze-string>
		</xsl:if>
		<xsl:if test="matches($style, 'url')">
			<param name="color">
				<color><r>0.5</r><g>0.5</g><b>0.5</b><a>0.5</a>	</color>
			</param>
		</xsl:if>
	</xsl:template>

	<xsl:function name="math:resolve_transform">
		<xsl:param name="transform"/>
			<xsl:variable name="stripped" select="replace(replace($transform, 'translate\(', 'X(1,0,0,1,'), 'matrix', 'X')"/>
			<xsl:analyze-string select="concat('X(1,0,0,1,0,0)', $stripped)" regex="(.*)X\((-?[\d.]+),(-?[\d.]+),(-?[\d.]+),(-?[\d.]+),(-?[\d.]+),(-?[\d.]+)\)[^X]*X\((-?[\d.]+),(-?[\d.]+),(-?[\d.]+),(-?[\d.]+),(-?[\d.]+),(-?[\d.]+)\).*">
				<xsl:non-matching-substring>
					<xsl:sequence select="(1,0,0,1,0,0)"/>
				</xsl:non-matching-substring>
				<xsl:matching-substring>
					<xsl:variable name="a2" select="xs:float(regex-group(8))"/>
					<xsl:variable name="b2" select="xs:float(regex-group(9))"/>
					<xsl:variable name="c2" select="xs:float(regex-group(10))"/>
					<xsl:variable name="d2" select="xs:float(regex-group(11))"/>
					<xsl:variable name="e2" select="xs:float(regex-group(12))"/>
					<xsl:variable name="f2" select="xs:float(regex-group(13))"/>
					<xsl:variable name="a1" select="xs:float(regex-group(2))"/>
					<xsl:variable name="b1" select="xs:float(regex-group(3))"/>
					<xsl:variable name="c1" select="xs:float(regex-group(4))"/>
					<xsl:variable name="d1" select="xs:float(regex-group(5))"/>
					<xsl:variable name="e1" select="xs:float(regex-group(6))"/>
					<xsl:variable name="f1" select="xs:float(regex-group(7))"/>
					<xsl:variable name="p" select="($a1*$a2+$c1*$b2,$b1*$a2+$d1*$b2,$a1*$c2+$c1*$d2,$b1*$c2+$d1*$d2,$a1*$e2+$c1*$f2+$e1,$b1*$e2+$d1*$f2+$f1)"/>
					<xsl:variable name="remainder" select="replace(regex-group(1), 'X\(1,0,0,1,0,0\)', '')"/>
					<xsl:choose>
						<xsl:when test="matches($remainder, 'X')">
							<xsl:variable name="recursion" select="concat($remainder, 'X(', $p[1], ',', $p[2], ',', $p[3], ',', $p[4], ',', $p[5], ',', $p[6], ')')"/>
							<xsl:sequence select="math:resolve_transform($recursion)"/>
						</xsl:when>
						<xsl:otherwise>
							<xsl:sequence select="$p"/>
						</xsl:otherwise>
					</xsl:choose>
				</xsl:matching-substring>
			</xsl:analyze-string>
	</xsl:function>

	<xsl:function name="math:hex_to_color" as="xs:float">
		<xsl:param name="hex"/>
		<xsl:value-of select="math:power(xs:float(string-length(substring-before('0123456789abcdef', substring($hex,1,1))) * 16 + string-length(substring-before('0123456789abcdef', substring($hex,2,1)))) div 255, 2.2)"/>
	</xsl:function>

	<xsl:function name="math:units_to_px" as="xs:float">
		<xsl:param name="size"/>
		<xsl:analyze-string select="$size" regex="^([-\d.]+)([a-z%]*)$">
			<xsl:matching-substring>
				<xsl:variable name="factor">
					<xsl:choose>
						<xsl:when test="regex-group(2) = 'pt'">1.25</xsl:when>
						<xsl:when test="regex-group(2) = 'em'">16</xsl:when>
						<xsl:when test="regex-group(2) = 'mm'">3.54</xsl:when>
						<xsl:when test="regex-group(2) = 'pc'">15</xsl:when>
						<xsl:when test="regex-group(2) = 'cm'">35.43</xsl:when>
						<xsl:when test="regex-group(2) = 'in'">90</xsl:when>
						<xsl:otherwise>1</xsl:otherwise>
					</xsl:choose>
				</xsl:variable>
				<xsl:value-of select="xs:float($factor) * xs:float(regex-group(1))"/>
			</xsl:matching-substring>
		</xsl:analyze-string>
	</xsl:function>
</xsl:stylesheet>
</nowiki></pre>

== Option 2 ==

This is a C program by akagogo that uses libxml to convert SVG to Synfig format.

http://none.carlos.googlepages.com/svgtosif.zip

=== Installation ===

Just make the usual '''./configure && make && sudo make install'''

=== Usage ===

The SVG files needs to be inside a folder called '''data'''. You have to run the command from the parent directory, but just using the name file as the command argument. So if:

* You are in folder "/example" you have to create a folder called "/example/data" and put the file "file.svg" there.
* Now you execute '''svgtosif file.svg''' when you got "/example" as your current working directory.

As you can see, this is really not to friendly. To quickly fix the problem, use the following bash script:

 #!/bin/bash
 mkdir data
 cp "$1" data/
 /usr/local/bin/svgtosif "$1"
 NAME=`echo "$1" | cut -d "." -f 1`
 cp "data/$NAME.sif" .
 rm data/*
 rmdir data
 echo "Conversion complete!"

Put those lines in a file named "svg2sif" (name it as you want, but avoid using "svgtosif") and put it in a PATH directory (suggest /usr/bin). Then:
 chmod +x /usr/bin/svg2sif 

Thats all. Now use:
 svg2sif <file.svg>

and you will get a .sif file in the same folder you are working in.
