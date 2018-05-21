{% assign junit-team = 'https://github.com/junit-team' %}
{% assign junit5-samples-repo = junit-team | append: '/junit5-samples' %}

{% assign javadoc-root = 'https://junit.org/junit5/docs/' | append: docs-version | append: '/api' %}                

{% assign snapshot-repo = 'https://oss.sonatype.org/content/repositories/snapshots' %}


{% assign params-provider-package = '[org.junit.jupiter.params.provider](' | append: javadoc-root | append: '/org/junit/jupiter/params/provider/package-summary.html)' %}
{% assign StackOverflow = '[Stack Overflow](https://stackoverflow.com/questions/tagged/junit5)' %}
{% assign Gitter = '[Gitter](https://gitter.im/junit-team/junit5)' %}                     


{% assign TestEngine = '[TestEngine](' | append: javadoc-root | append: '/org/junit/platform/engine/TestEngine.html)' %}
