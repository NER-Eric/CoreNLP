---
title: CorefAnnotator 
keywords: coref
permalink: '/coref.html'
---

## Description

Implements both pronominal and nominal coreference resolution. The entire coreference graph (with head words of mentions as nodes) is saved in CorefChainAnnotation. 

| Property name | Annotator class name | Generated Annotation |
| --- | --- | --- |
| dcoref | DeterministicCorefAnnotator | CorefChainAnnotation | 
| coref | CorefAnnotator              | CorefChainAnnotation |

## Options

* coref.doClustering: if using statistical mode, use clustering algorithm for coreference ; clustering is slower but more accurate
* coref.md.type: determine which mention detector to use: rule, dependency, or hybrid
* coref.mode: determine which coreference system to use: statistical or hybrid
* dcoref.sievePasses: list of sieve modules to enable in the system, specified as a comma-separated list of class names. By default, this property is set to include: "edu.stanford.nlp.dcoref.sievepasses.MarkRole, edu.stanford.nlp.dcoref.sievepasses.DiscourseMatch, edu.stanford.nlp.dcoref.sievepasses.ExactStringMatch, edu.stanford.nlp.dcoref.sievepasses.RelaxedExactStringMatch, edu.stanford.nlp.dcoref.sievepasses.PreciseConstructs, edu.stanford.nlp.dcoref.sievepasses.StrictHeadMatch1, edu.stanford.nlp.dcoref.sievepasses.StrictHeadMatch2, edu.stanford.nlp.dcoref.sievepasses.StrictHeadMatch3, edu.stanford.nlp.dcoref.sievepasses.StrictHeadMatch4, edu.stanford.nlp.dcoref.sievepasses.RelaxedHeadMatch, edu.stanford.nlp.dcoref.sievepasses.PronounMatch".  The default value can be found in Constants.SIEVEPASSES.
* dcoref.demonym: list of demonyms from <a href="http://en.wikipedia.org/wiki/List_of_adjectival_forms_of_place_names">http://en.wikipedia.org/wiki/List_of_adjectival_forms_of_place_names</a>. The format of this file is: location TAB singular gentilic form TAB plural gentilic form, e.g., "Algeria Algerian Algerians".
* dcoref.animate and dcoref.inanimate: lists of animate/inanimate words, from (Ji and Lin, 2009). The format is one word per line.
* dcoref.male, dcoref.female, dcoref.neutral: lists of words of male/female/neutral gender, from (Bergsma and Lin, 2006) and (Ji and Lin, 2009). The format is one word per line.
* dcoref.plural and dcoref.singular: lists of words that are plural or singular, from (Bergsma and Lin, 2006). The format is one word per line. All the above dictionaries are already set to the files included in the stanford-corenlp-models JAR file, but they can easily be adjusted to your needs by setting these properties.
* dcoref.maxdist: the maximum distance at which to look for mentions.  Can help keep the runtime down in long documents.
* oldCorefFormat: produce a CorefGraphAnnotation, the output format used in releases v1.0.3 or earlier.  Note that this uses quadratic memory rather than linear.

## Overview

There are several settings which can effect the speed and accuracy of coreference resolution.
Some methods require both constituency and dependency parses, while others simply need the dependency parse.
The following tables give an overview of some of the possibilities. 

* the F1 scores are on the 2012 CoNLL evaluation data
* the speed measurements were recorded on a 2014 Macbook Pro with a 2.5 GHz Intel Core i7 processor

| Annotator | Language | Coreference/MD Modes | Parse Requirements | F1 score |
| :--- | :--- | :--- | :--- | :--- | :--- |
| coref | en | statistical/rule | constituency and dependency | 63.61 |
| coref | en | statistical/dependency | dependency | 56.05 |
| dcoref | en | N/A | constituency and dependency | 55.59 |
| coref | zh | hybrid/rule | constituency and dependency | 53.18 |

| Annotator | Language | Coreference/MD Modes | Parsing Speed | Coref Speed |
| :--- | :--- | :--- | :--- | :--- |
| coref | en | statistical/rule | 4.45 seconds per doc | 3.50 seconds per doc | 
| coref | en | statistical/dependency | .049 seconds per doc | 1.03 seconds per doc |
| dcoref | en | N/A | 4.45 seconds per doc | .123 seconds per doc |
| coref | zh | hybrid/rule | 19.1 seconds per doc | .044 seconds per doc |

## Usage

### Command Line

Run statistical coref to maximize F1 on the 2012 CoNLL data set

```bash
java -Xmx5g -cp stanford-corenlp-3.6.0.jar:stanford-corenlp-models-3.6.0.jar edu.stanford.nlp.pipeline.StanfordCoreNLP -annotators tokenize,ssplit,pos,lemma,ner,parse,mention,coref -file example_file.txt
```

Run statistical coref for maximum speed

```bash
java -Xmx5g -cp stanford-corenlp-3.6.0.jar:stanford-corenlp-models-3.6.0.jar edu.stanford.nlp.pipeline.StanfordCoreNLP -annotators tokenize,ssplit,pos,lemma,ner,depparse,mention,coref -file example_file.txt 
-coref.md.type dependency -coref.doClustering false 
```

Run deterministic coref

```bash
java -Xmx3g -cp stanford-corenlp-3.6.0.jar:stanford-corenlp-models-3.6.0.jar edu.stanford.nlp.pipeline.StanfordCoreNLP -annotators tokenize,ssplit,pos,lemma,ner,parse,dcoref -file example_file.txt
```

Run Chinese coref

```bash
java -Xmx4g -cp stanford-corenlp-3.6.0.jar:stanford-chinese-corenlp-2015-12-07-models.jar edu.stanford.nlp.pipeline.StanfordCoreNLP -file example_file.txt -props edu/stanford/nlp/hcoref/properties/zh-dcoref-default.properties
```

### API

Accessing coref and mention information from an Annotation

```java
import edu.stanford.nlp.hcoref.CorefCoreAnnotations;
import edu.stanford.nlp.hcoref.data.CorefChain;
import edu.stanford.nlp.hcoref.data.Mention;
import edu.stanford.nlp.ling.CoreAnnotations;
import edu.stanford.nlp.pipeline.Annotation;
import edu.stanford.nlp.pipeline.StanfordCoreNLP;
import edu.stanford.nlp.util.CoreMap;

import java.util.Properties;

public class CorefExample {

  public static void main(String[] args) throws Exception {

    Annotation document = new Annotation("Barack Obama was born in Hawaii.  He is the president.  Obama was elected in 2008.");
    Properties props = new Properties();
    props.setProperty("annotators", "tokenize,ssplit,pos,lemma,ner,parse,mention,coref");
    StanfordCoreNLP pipeline = new StanfordCoreNLP(props);
    pipeline.annotate(document);
    System.out.println("---");
    System.out.println("coref chains");
    for (CorefChain cc : document.get(CorefCoreAnnotations.CorefChainAnnotation.class).values()) {
      System.out.println("\t"+cc);
    }
    for (CoreMap sentence : document.get(CoreAnnotations.SentencesAnnotation.class)) {
      System.out.println("---");
      System.out.println("mentions");
      for (Mention m : sentence.get(CorefCoreAnnotations.CorefMentionsAnnotation.class)) {
        System.out.println("\t"+m);
       }
    }
  }
}
```

## More information 

For more details on the underlying coreference resolution algorithm, see [this page](http://nlp.stanford.edu/software/dcoref.shtml).