Javascript test tools
=====================

Django.js provide tools for easy javascript testing.

Views
-----
Django.js provides base views for javascript testing.
Instead of writing a full view each time you need a Jasmine or a QUnit test view, simply use the provided ``JasmineView`` and ``QUnitView`` and add them to your test_urls.py:


.. code-block:: python

    from django.conf.urls import patterns, url, include

    from djangojs.views import JasmineView, QUnitView

    urlpatterns = patterns('',
        url(r'^jasmine$', JasmineView.as_view(js_files='js/specs/*.specs.js'), name='my_jasmine_view'),
        url(r'^qunit$', QUnitView.as_view(js_files='js/tests/*.tests.js'), name='my_qunit_view'),
    )

These views extends the Django TemplateView so you can add extra context entries and you can customize the template by extending them.

.. code-block:: python

    from django.conf.urls import patterns, url, include

    from djangojs.views import JasmineView, QUnitView


    class MyJasmineView(JasmineView):
        js_files = 'js/test/*.specs.js'

    class MyQUnitView(QUnitView):
        js_files = 'js/test/*.test.js'
        template_name = 'my-qunit-runner.html'

        def get_context_data(self, **kwargs):
            context = super(MyQUnitView, self).get_context_data(**kwargs)
            context['form'] = TestForm()
            return context

    urlpatterns = patterns('',
        url(r'^jasmine$', MyJasmineView.as_view(), name='my_jasmine_view'),
        url(r'^qunit$', MyQUnitView.as_view(), name='my_qunit_view'),
    )

Two extensible test runner templates are provided:

- ``djangojs/jasmine-runner.html`` for jasmine tests
- ``djangojs/qunit-runner.html`` for QUnit tests

Both provides a ``js_content`` block and a ``body_content`` block.

.. code-block:: html+django

    {% extends "djangojs/qunit-runner.html" %}

    {% block js_content %}
    {% load js %}
    {% js "js/tests/my.tests.js" %}
    {% endblock %}

    {% block body_content %}
      <form id="test-form" action="{% url test_form %}" method="POST" style="display: none;">
        {{csrf_token}}
        {{form}}
      </form>
    {% endblock %}

You can inspect django.js own test suites on github.

Test cases
----------

A Phantom.js test runner is provided with Jasmine/QUnit support.
To use it with the previously defined views, just call the ``run_jasmine()`` or ``run_qunit()`` methods:

.. code-block:: python

    class JsTests(JsTestCase):
        urls = 'myapp.test_urls'

        def test_jasmine_suite(self):
            '''It should run its its own Jasmine test suite'''
            self.run_jasmine('my_jasmine_view', title='Jasmine Test Suite')

        def test_qunit_suite(self):
            '''It should run its its own QUnit test suite'''
            self.run_qunit('my_qunit_view', title='QUnit Test Suite')

The verbosity is automatically adjusted with the ``-v/--verbosity`` parameter from the ``manage.py test`` command line.


.. warning::

    Phantom.js is required to use this feature and should be on your ``$PATH``.